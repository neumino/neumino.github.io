---
layout: post
category : Geek
tags : [rethinkdbdash, rethinkdb, node]
title: Paving the way for the new Nodejs
---
{% include JB/setup %}

RethinkDB works really hard to build the best database, which includes delivering [the
best user experience as possible](https://github.com/rethinkdb/rethinkdb/issues/1000).  
For developers, RethinkDB built ReQL, a unified chainable dynamic query language that lets
developers write queries __in a breeze__. If you hate SQL or enjoy writing MongoDB
queries, you will love ReQL.

The JavaScript driver, while providing a great implementation of ReQL, still had to
deal with callbacks making it somehow a little cucumbersome to use.

Node.js recently [announced](http://blog.nodejs.org/2014/01/16/nodejs-road-ahead/) their
next stable release as being imminent. What most Nodejs developpers expect from this
release is probably the possibility to use generators. As a result, I took this
opportunity to write a new driver from scratch.

This driver was written taking into consideration what people were building with the
current one.

- __Promises__  
A few projects wrapped the RethinkDB driver with different libraries --
[rql-promise](https://npmjs.org/package/rql-promise),
[reql-then](https://npmjs.org/package/reql-then)  
This driver provides native promises.

- __Connection pool__  
Many ORMs ([reheat](https://npmjs.org/package/reheat),
[thinky](https://npmjs.org/package/thinky), etc.) and connectors
([sweets-nougat](https://npmjs.org/package/sweets-nougat),
[waterline-rethinkdb](https://npmjs.org/package/waterline-rethinkdb), etc.)
are implementing their own connection pool on top of the driver.  
This driver provides a native connection pool without having to manually acquire and
release a connection.

- __More accessible__  
The current JavaScript driver is buried in RethinkDB main repository, is written in
CoffeeScript and its tests are designed to run for all official drivers. In the end, it
is hard to contribute to the driver.  
This driver has its own repository, is written in full JavaScript, and tests are run
with [mocha](http://visionmedia.github.io/mocha/)
(and [on wercker](https://app.wercker.com/#applications/52dffe8ba4acb3ef16010ef8)).


The result of all these considerations is
[rethinkdbdash](https://github.com/neumino/rethinkdbdash), which is now, as far as I
know, feature complete and working.  

So how is it better? Well, here is for example how you would fetch all the posts
of your blog with their comments.

```js
function* getPosts() {
  try{
    var cursor = yield r.db("blog").table("posts").map(function(post) {
      return post.merge({
        comments: r.db("blog").table("comments")
          .filter({idPost: post("id")})
          .coerceTo("array")
      })
    }).run()
    var results = yield cursor.toArray();
    this.body = JSON.stringify(results);
  }
  catch(e) {
    this.status = 500;
    this.body = e.message || http.STATUS_CODES[this.status];
  }
}
```
What is awesome here, is:

- There is no callback.
- Only one query is executed, the anonymous function is compiled and sent to the server.
- You do not need to open/provide/close a connection.

Take a look at the usual
[Todo example](https://github.com/neumino/rethinkdbdash-examples/tree/master/todo)
built with using [AngularJS](http://angularjs.org/), [Koa](https://github.com/koajs/koa)
and [Rethinkdbdash](https://github.com/neumino/rethinkdbdash).

The only thing for this driver to hopefully become mainsteam is the release of Nodejs
0.12.0, since if you want to use it now, you have to build Nodejs 0.11.10 from source (some
symbols are missing in the unstable binaries).


[Feedback](https://twitter.com/neumino),
[pull requests](https://github.com/neumino/rethinkdbdash/pulls)
and [comments](...HN Thread...) are welcome!  
