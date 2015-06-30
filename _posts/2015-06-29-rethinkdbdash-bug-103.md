---
layout: post
category : Geek
tags : [rethinkdbdash, nodejs, bug]
title: "Rethinkdbdash bug #103"
---
{% include JB/setup %}


I just fixed a sneaky but in rethinkdbdash and thought it would be worth
to quickly write about it.


Here is the broken snippet:

```js
// Opening a socket
self.connection = net.connect({
  host: self.host,
  port: self.port,
  family: family
});

self.connection.on('end', function(error) {
  // ...
});
self.connection.on('close', function(error) {
  // ...
});
self.connection.setNoDelay();
self.connection.on('connect', function() {
  // Do the handshake
  // var initBuffer = ...
  // var lengthBuffer = ...
  // var authBuffer ...
  // var protocolBuffer = ...
  self.connection.write(Buffer.concat([initBuffer, lengthBuffer, authBuffer, protocolBuffer]));
});
self.connection.once('error', function(error) {
  // ...
});
self.connection.once('end', function() {
  self.open = false;
});

self.connection.on('data', function(buffer) {
  // Handle handshake and responses
});
```

I have cleaned a bit the code, but the main idea is to open a TCP connection,
send a "handshake" and listen for data (the response of the handshake and the response
for the queries we will send).

This code however can throw with

```
Process exit at 2015-06-27T15:15:18.897Z
events.js:141
      throw er; // Unhandled 'error' event
            ^
Error: write after end
    at writeAfterEnd (_stream_writable.js:158:12)
    at Socket.Writable.write (_stream_writable.js:203:5)
    at Socket.write (net.js:615:40)
    at /home/u/app/node_modules/rethinkdbdash/lib/connection.js:81:23
    at Object.tryCatch (/home/u/app/node_modules/rethinkdbdash/lib/helper.js:156:3)
    at Socket.<anonymous> (/home/u/app/node_modules/rethinkdbdash/lib/connection.js:80:12)
    at emitNone (events.js:72:20)
    at Socket.emit (events.js:166:7)
    at TCPConnectWrap.afterConnect [as oncomplete] (net.js:1029:10)
```

The reason is that `self.connection.write` is not safe there.

The main problem is that the TCP connection can be already closed when you enter the callback for the
`connect` event. In this case, we will try to write on the socket, but this will
[immediately emit](https://github.com/joyent/node/blob/ef4344311e19a4f73c031508252b21712b22fe8a/lib/_stream_writable.js#L171) an
error. Because we bind the listener for `error` after the one for `connect`, we end up with an error that nothing
will catch that will eventually crash the worker.

The solution is simply to bind your listener for `error` before trying to write anything on the socket. A better solution
would probably be for node to emit the event at the next tick like mentionned in the
[TODO](https://github.com/joyent/node/blob/ef4344311e19a4f73c031508252b21712b22fe8a/lib/_stream_writable.js#L170).
