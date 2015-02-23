---
layout: post
category : Geek
tags : [nodejs, iojs, writable, transform, streams, rethinkdbdash]
title: "Batching operations in Node.js writable streams"
---
{% include JB/setup %}

I recently implemented writable and transform [streams](nodejs.org/api/stream.html) in [rethinkdbdash](https://github.com/neumino/rethinkdbdash).
Importing a file to a [RethinkDB](http://rethinkdb.com) table is now as simple as piping a
[Readable](http://nodejs.org/api/stream.html#stream_class_stream_readable) stream into a
[Writable](http://nodejs.org/api/stream.html#stream_class_stream_writable) one.
The constraints for the implementation were:

- Insert documents in order (first in, first written).
- Insert documents in batch of at most `highWaterMark` documents.

I somehow couldn't find anything about this use case, and now that I have
implemented these streams, I kind of understand why. Basically the current API
is not designed for batching operations - even [Transform](http://nodejs.org/api/stream.html#stream_class_stream_transform)
streams that support buffering (with `_flush`) behave poorly when it comes
to efficiently batching operations.

Let's first look at the [API](http://nodejs.org/api/stream.html#stream_class_stream_writable_1)
for Writable streams. We have to implement `writable._write(value, encoding, done)`.
In rethinkdbdash case, we want to insert documents, so the stream are created in `objectMode`;
`value` is one document, `encoding` is not relevant, and `done` is to be called when we are done
processing the supplied document.

A basic implementation would be:

```js
var Writable = require('stream').Writable;
var r = require('rethinkdbdash')();

function WritableStream(table) {
  this._table = table; // The RethinkDB table to insert in

  Writable.call(this, {
      objectMode: true,
      highWaterMark: this._highWaterMark
  });
}

WritableStream.prototype._write = function(value, encoding, done) {
  r.table(this._table).insert(value).run().then(function(result) {
    done();
  }).error(done);
}
```

Let's consider how to batch inserts now.
Looking at [the implementation](https://github.com/joyent/node/blob/v0.12.0-release/lib/_stream_writable.js)
of `Writable` streams in Node.js, a `Writable` streams has an internal buffer implemented as a linked list
(since 0.12 - see the relevant [pull request](https://github.com/joyent/node/pull/8826)), and you can access the
last element of this list via [stream._writableState.lastBufferRequest](https://github.com/joyent/node/blob/v0.12.0-release/lib/_stream_writable.js#L115).

So what we can do is look at the current `value` we are processing, check if it is the same as
the last available value in the internal buffer and from here:

- just call `done` to keep buffering if there is more data available
- flush what we have if there is nothing more in the internal buffer.

So the implementation becomes:

```js
var Writable = require('stream').Writable;
var r = require('rethinkdbdash')();

function WritableStream(table) {
  this._table = table; // The RethinkDB table to insert in
  this._cache = []; // Current cached documents that will be saved at the next insertion
  this._highWaterMark = 16; // The size of our cache and or the internal buffer

  Writable.call(this, {
      objectMode: true,
      highWaterMark: this._highWaterMark
  });
}

WritableStream.prototype._write = function(value, encoding, done) {
  this._cache.push(value);

  if ((this._writableState.lastBufferedRequest != null) &&
      (this._writableState.lastBufferedRequest.chunk !== value)) {
    // There is more data available
    if (this._cache.length === this._highWaterMark) {
      // Our cache is full, flush documents
      this._insert(done)
    }
    else {
      // We have more data coming, let's buffer more
      done();
    }
  }
  else {
    // There is NO more data available, flush what we have
    this._insert(done);
  }
}

WritableStream.prototype._insert = function(done) {
  var values = this._cache;
  this._cache = [];
  r.table(this.table).insert(values).run().then(function(result) {
    done();
  }).error(done);
}
```


While this implementation looks fine, it suffers from the fact that when we are inserting, the
stream is not re-filling its internal buffer. If we inspect the size of `this.cache` before each insert,
when the Writable streams is provided a fast Readable stream, we will see roughly a
sequence like `1`, `highWaterMark-1`, `1`, `highWaterMark-1`, `1`, `highWaterMark-1` etc.

From here things become tricky. Trying to call `done` just after calling `this._insert` to
attempt re-buffering is actually not enough. From an implementation point of view, `done` should be
called immediately after `this._insert` as long as there is more data in the cache. If
there is no more data available and if we are already inserting, we have to keep a reference of `done`
(in `this._pendingCallback` in the implementation below) and call it once the current
insertion is complete.

One problem that surfaces after implementing this solution is that if you have
a fast stream piping into your writable stream, the internal buffer of the
writable stream will not buffer fast enough; you still have that sequence of `1` and `highWaterMark-1`.
The only way I found to have optimal buffering in case of an efficient input is to move
the call of `_write` in the call stack.

The implementation looks like this one.

```js
var Writable = require('stream').Writable;
var r = require('rethinkdbdash')();

function WritableStream(table, options, connection) {
  this._table = table; // The RethinkDB table to insert in
  this._cache = []; // Current cached documents that will be saved at the next insertion
  this._pendingCallback = null; // The callback to call when we are done inserting
  this._inserting = false; // Whether an insertion is happening
  this._delayed = false; // Whether the current call to _next was moved in the call stack or not
  this._highWaterMark = options.highWaterMark || 16; // The size of our cache and or the internal buffer

  Writable.call(this, {
    objectMode: true,
    highWaterMark: this._highWaterMark
  });
};
util.inherits(WritableStream, Writable);

WritableStream.prototype._write = function(value, encoding, done) {
  this._cache.push(value);
  this._next(value, encoding, done);
}

// Everytime we want to insert but do not have a full buffer,
// we recurse with setImmediate to give a chance to the input
// stream to push a few more elements
WritableStream.prototype._next = function(value, encoding, done) {
  if ((this._writableState.lastBufferedRequest != null) &&
      (this._writableState.lastBufferedRequest.chunk !== value)) {
    // There's more data to buffer
    if (this._cache.length < this._highWaterMark) {
      this._delayed = false;
      // Call done now, and more data will be put in the cache
      done();
    }
    else {
      if (this._inserting === false) {
        if (this._delayed === true) {
          this._delayed = false;
          // We have to flush
          this._insert();
          // Fill the buffer while we are inserting data
          done();
        }
        else {
          var self = this;
          this._delayed = true;
          setImmediate(function() {
              self._next(value, encoding, done);
          })
        }
      }
      else {
        this._delayed = false;
        // to call when we are dong inserting to keep buffering
        this._pendingCallback = done;
      }
    }
  }
  else { // We just pushed the last element in the internal buffer
    if (this._inserting === false) {
      if (this._delayed === true) {
        this._delayed = false;
        // to call when we are dong inserting to maybe flag the end
        // We cannot call done here as we may be inserting the last batch
        this._pendingCallback = done;
        this._insert();
      }
      else {
        var self = this;
        this._delayed = true;
        setImmediate(function() {
          self._next(value, encoding, done);
        })
      }
    }
    else {
      this._delayed = false;
      // We cannot call done here as we may be inserting the last batch
      this._pendingCallback = done;
    }
  }
}

WritableStream.prototype._insert = function() {
  var self = this;
  self._inserting = true;

  var cache = self._cache;
  self._cache = [];

  self._table.insert(cache).run().then(function(result) {
    self._inserting = false;
    if (result.errors > 0) {
      self.emit('error', new Error("Failed to insert some documents:"+JSON.stringify(result, null, 2)));
    }
    if (typeof self._pendingCallback === 'function') {
      var pendingCallback = self._pendingCallback;
      self._pendingCallback = null;
      pendingCallback();
    }
  }).error(function(error) {
    self._inserting = false;
    self.emit('error', error);
  });
}
```

This may be a bit hard to digest, but when we have nothing to add in our cache, we flip `this._delayed` to
`true` and call the same function (`_next`) with `setImmediate`. It is only if we end up in the same situation that we
will insert an incomplete batch. [Tests](https://github.com/neumino/rethinkdbdash/blob/93fba3db1ac54f80a5f9a15cc979506b92081a56/test/writable-stream.js#L47)
show a better flow of data (a sequence of `highWaterMark` values instead of `1` and `highWaterMark-1`).

Transform streams can be implemented slightly in a more efficient way thanks to the `_flush` method; we
can always call `done` after `insert` in an attempt to buffer more incoming data,
but as far as I can tell, moving `_write` in the call stack is also required for a better
flow.

The API for Node.js streams is still tagged as `unstable`, so hopefully the API will
become more friendly for the use cases similar to the one described here.

Questions? Feedback? Shoot me a mail at [orphee@gmail.com](mailto:orphee@gmail.com)
or ping me on Twitter [@neumino](https://twitter.com/neumino)

_Note_: Because the internal buffer changed data structure, the implementation showed above
does not work that well in Node 0.10.
