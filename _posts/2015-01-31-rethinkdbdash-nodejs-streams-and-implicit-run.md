---
layout: post
category : Geek
tags : [rethinkdbdash, rethinkdb, driver, nodejs, iojs]
title: "Rethinkdbdash 1.16: Node.js streams, implicit run and more"
---
{% include JB/setup %}

_Note_: Never heard of RethinkDB? You may want to read about [where it's going](http://rethinkdb.com/blog/realtime-web/).

[Rethinkdbdash](https://github.com/neumino/rethinkdbdash) is an experimental
yet stable Node.js driver for [RethinkDB](http://rethinkdb.com). It started as a playground to test
promises and connection pools, and still strives today to provide a better
experience for developers.

A new version just got released today, and besides an update to support the new 
[RethinkDB 1.16](http://www.rethinkdb.com/blog/1.16-release/) commands,
it comes with a few new features inspired from
[@jonathanong](https://github.com/jonathanong)'s writing about an
[ideal-database-client](https://github.com/jonathanong/ideal-database-client).

This blog post aims to describe the main differences between rethinkdbdash
and the official driver.

### Support for Node.js streams

One of the new feature in rethinkdbdash is the support for [Node.js streams](http://nodejs.org/api/stream.html).
You can retrieve a stream with the synchronous method `toStream`.

```js
// Create a transform stream that will convert data to a string
var stream = require('stream')
var stringifier = new stream.Transform();
stringifier._writableState.objectMode = true;
stringifier._transform = function (data, encoding, done) {
  this.push(JSON.stringify(data));
  this.push('\n');
  done();
}

// Create a writable stream
var fs = require('fs');
var file = fs.createWriteStream('result.txt');

var r = require('rethinkdbdash')();
// Pipe the data to the file through stringifier
r.table("data").toStream()
    .pipe(stringifier)
    .pipe(file);
```

### Optional run

ReQL is composed of chainable methods, requiring the developer to call the `run`
command at the end when the query is complete and ready to be sent to the server.
By implementing `then` and `catch` as shortcuts for `run().then` and `run().catch`,
any query will be executed when called with `yield` without the need to call `run`.

You can now write queries like this:

```js
var bluebird = require('bluebird');
var r = require('rethinkdbdash')();
bluebird.coroutine(function*() {
    var result = yield r.table("users").insert({id: 1, name: "Michel", age: 28});
    assert.equal(result.inserted, 1);

    result = yield r.table("users").get(1);
    assert.deepEqual(result, {id: 1, name: "Michel", age: 28});

    result = yield r.table("users").get(1).update({name: "John"});
    assert.equal(result.replaced, 1);

    result = yield r.table("users").get(1);
    assert.deepEqual(result, {id: 1, name: "John", age: 28});

    result = yield r.table("users").get(1).update(function(user) {
        return { isAdult: user("age").gt(18) }
    });
    assert.equal(result.replaced, 1);

    result = yield r.table("users").get(1);
    assert.deepEqual(result, {id: 1, name: "John", age: 28, isAdult: true});
});
```

### A single point of failure

Rethinkdbdash has been shipped with a connection pool since its first version. This
connection pool is managed by the driver itself, and users do not have to create or manage
connections themselves. This has two purposes:

- Remove repetitive code.
- Provide a unique point of failure. Consider for example this snippet:

```js
var handleError = function(error) {
    res.status(500).json(error: JSON.stringify(error));
};

r.connect().bind({}).then(function(connection) {
    this._connection = connection;
    this._connection.on('error', handleError);
    return r.table("users").get("orphee@gmail.com").run()
}).then(function(user) {
    res.send(JSON.stringify(user));
    return this._connection.close();
}).then(function() {
    this._connection.off('error', handleError);
}).error(handleError);
```

The problem in this code is that the connection can emit an error at any time, meaning
that you have multiple point of failures (the connection and the query itself). 
In the end, depending on your case, you may have to deal with race conditions. Typically here,
`handleError` may be called twice.

With rethinkdbdash, network errors are caught by the driver and bubbled to
the relevant queries. Compare the previous code with:

```js
r.table("users").get("orphee@gmail.com").run().then(function(user) {
    res.send(JSON.stringify(user));
}).error(function(error) {
    // Catch error returned by the driver
    // Catch network errors
    res.status(500).json(error: JSON.stringify(error));
});
```



### Like what you read?
Give it a spin:

- [Install](http://www.rethinkdb.com/docs/install/) RethinkDB
- Install rethinkdbdash with `npm install rethinkdbdash`
- Checkout the [examples](https://github.com/neumino/rethinkdbdash-examples)
- Questions? Feedback? Shoot me a mail at [orphee@gmail.com](mailto:orphee@gmail.com)
or ping me on Twitter [@neumino](https://twitter.com/neumino)

_Note about streams_: `null` values are currently silently dropped from streams
as in `objectMode`, a `null` value means the end of the stream.

_Note_: `toStream` was added in rethinkdbdash 1.16.4. The argument `{stream: true}` has
been deprecated.
