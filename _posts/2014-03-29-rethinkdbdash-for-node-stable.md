---
layout: post
category : Geek
tags : [rethinkdb, nodejs]
title: Rethinkdbdash for Node.js 0.10.26
---
{% include JB/setup %}

I just released two packages of [rethinkdbdash](https://github.com/neumino/rethinkdbdash)

- `rethinkdbdash` for Node.js `0.10.26`
- `rethinkdbdash-unstable` for Node.js `0.11.10` (and `0.11.9`)


I wrote `rethinkdbdash` two months ago to improve the syntax of ReQL in the Node.js driver
by providing

- promises (and testing them
[with generators](http://blog.justonepixel.com/geek/2014/01/27/paving-the-way-for-the-new-nodejs/))
- a native/automatic connection pool


While you cannot use generators with the stable version of Node.js, the connection pool is a
useful feature since you never have to deal with connections.


For those who want to know what the syntax looks like, here it is:

```js
var r = require('rethinkdbdash')();

r.table("comments").get("eef5fa0c").run().then(function(result) {
    console.log(result);
}).error(function(error) {
    console.log(error);
})
```

Compared to the one with the official driver:

```js
var r = require('rethinkdb');

r.connect({}, function(error, connection) {
    r.table("comments").get("eef5fa0c").run(function(error, result) {
        if (err) {
            console.log(error)
        }
        else {
            console.log(result);
        }
        connection.close();
    })
})
```



_Note_: If you were using `rethinkdbdash` with Node `0.11.10`, please switch to `rethinkdbdash-unstable`.
