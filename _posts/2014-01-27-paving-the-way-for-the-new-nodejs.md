---
layout: post
category : Geek
tags : [rethinkdbdash, rethinkdb, node]
title: Paving the way for the new Nodejs
---
{% include JB/setup %}

[ReQL](http://rethinkdb.com/docs/introduction-to-reql/),
the RethinkDB Query Language, embeds into your programming
language, making it pleasant to write queries.  
The JavaScript driver is a great implementation of ReQL, but it forces users to deal
with callbacks making the code cumbersome.

Recently Nodejs [announced](http://blog.nodejs.org/2014/01/16/nodejs-road-ahead/) that
the next stable release, 0.12.0, is imminent. The biggest feature in Nodejs 0.12.0 most
developers are looking forward to is generators. Since generators remove the need for
cumbersome callback code, I decided to take the opportunity to write a new callback-free
RethinkDB driver from scratch.

I wrote this driver taking into consideration what people were building with the
current one.

- __Promises__  
A few projects wrapped the RethinkDB driver with different libraries --
[rql-promise](https://npmjs.org/package/rql-promise),
[reql-then](https://npmjs.org/package/reql-then)  
The new driver works with promises natively.

- __Connection pool__  
Many ORMs ([reheat](https://npmjs.org/package/reheat),
[thinky](https://npmjs.org/package/thinky), etc.) and connectors
([sweets-nougat](https://npmjs.org/package/sweets-nougat),
[waterline-rethinkdb](https://npmjs.org/package/waterline-rethinkdb), etc.)
are implementing their own connection pools on top of the driver.  
The new driver provides a native connection pool without having to manually acquire and
release a connection.

- __More accessible__  
The current JavaScript driver is buried in RethinkDB main repository, is written in
CoffeeScript and its tests are designed to run for all official drivers. In the end, it
is hard to contribute to the driver.  
The new driver has its own repository, is written in full JavaScript, and tests are run
with [mocha](http://visionmedia.github.io/mocha/)
(and [on wercker](https://app.wercker.com/#applications/52dffe8ba4acb3ef16010ef8)).


The result of all these considerations is
[rethinkdbdash](https://github.com/neumino/rethinkdbdash), which is now, as far as I
know, feature complete and working.  


How is [rethinkdash](https://github.com/neumino/rethinkdbdash) better than the official
JavaScript driver? Let's look at a concrete example.  

Suppose you have two tables with the following schemas:

```js
// Table posts
{
  id: String,
  title: String,
  content: String
}

// Table comments
{
  id: String,
  idPost: String, // id of the post
  comment: String
}
```

And suppose you want to fetch all the posts of your blog with their comments and retrieve
them with the following format:

```js
// Results
{
  id: String,
  title: String,
  content: String,
  comments: [ {id: String, idPost: String, comment: String}, ... ]
}
```

With ReQL you can retrieve everything in one query using a subquery[1].

This is how you currently do it with the old driver.

```js
var r = require("rethinkdb");
// Code for Express goes here...

function getPosts(req, res) {
  r.connect({}, function(error, connection) {
    if (error) return handleError(error);

    r.db("blog").table("posts").map(function(post) {
      return post.merge({
        comments: r.db("blog").table("comments").filter({idPost: post("id")}).coerceTo("array")
      })
    }).run(connection, function(error, cursor) {
      if (error) return handleError(error);

      cursor.toArray(function(error, results) {
        if (error) return handleError(error);

        res.send(JSON.stringify(results));
      })
    })
  });
}
```

Now look how wonderful the code looks with new driver.

```js
var r = require("rethinkdbdash")();
// Code for Koa goes here...

function* getPosts() {
  try{
    var cursor = yield r.db("blog").table("posts").map(function(post) {
      return post.merge({
        comments: r.db("blog").table("comments").filter({idPost: post("id")}).coerceTo("array")
      })
    }).run()
    var results = yield cursor.toArray();
    this.body = JSON.stringify(results);
  }
  catch(error) {
    return handleError(error);
  }
}
```
What is awesome here, is:

- There are no callbacks.
- You do not need to open/provide/close a connection.

Take a look at the usual
[Todo example](https://github.com/neumino/rethinkdbdash-examples/tree/master/todo)
built with [AngularJS](http://angularjs.org/), [Koa](https://github.com/koajs/koa)
end [Rethinkdbdash](https://github.com/neumino/rethinkdbdash).

We have to wait for the stable release of Node before the new driver can become
mainstream. In the meantime, you can build Nodejs 0.11.10 from source to play with
rethinkdbdash[2].  
Once people use it for a bit and it is better tested, it should become the official driver.


[Feedback](https://twitter.com/neumino),
[pull requests](https://github.com/neumino/rethinkdbdash/pulls)
and [comments](https://news.ycombinator.com/item?id=7146502) are welcome!  


------------

[1] The anonymous function in `map` is parsed and send to the server.  
Read [all about lambda functions in RethinkDB queries](http://www.rethinkdb.com/blog/lambda-functions/)
if that piques your interest.  

[2] You need to build from source or you may get errors like

```
node: symbol lookup error: /some/path/protobuf.node: undefined symbol: _ZN4node6Buffer3NewEm
```

