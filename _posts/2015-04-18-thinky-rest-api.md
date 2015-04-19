---
layout: post
category : Geek
tags : [nodejs, iojs, thinky, rest, api]
title: "On building a REST API with thinky"
---
{% include JB/setup %}

ReQL is an incredibly powerful beast, and thinky wields most of its power
by using the exact same API. While thinky can be used like classic ORMs
like mongoose, it is a shame to miss some of its really nice features.

This article describes how to build the thinky part of a REST API for Express,
and hopefully is the first of a serie that will showcase what thinky can do. This
article focuses only on thinky. If you want to learn how Express work, there is a
plethora of tutorial on the Internet.


I. _Create the model_

```js
var thinky = require('thinky')();
var type = thinky.type;
var r = thinky.r;

var User = thinky.createModel('User', {
  id: type.string(),
  name: type.string().required(),
  email: type.string().email().required(),
  createdAt: type.date().default(r.now())
});
```

The model has 4 fields:

- `id`: a simple string.
- `name`: a required string.
- `email`: a required string that should be a valid email.
- `createdAt`: the date at which the user was created.

The table will be automatically created under the hood. If you immediately fire
queries while the table is not ready, the queries will be queued.


II. _Insert a new user_

```js
function insert(request, response, next) {
  var user = new User(request.body);

  user.save().then(function(result) {
    res.send(JSON.stringify(result));
  }).error(function(error) {
    // Duplicate primary key, not valid document, network errors etc.
    response.send(500, {error: error.message}
  });
}
```

There are a few things to note here:

- The object `request.body` needs to only provide two fields `name` and `email`.
- The field `id` is the primary key and if `undefined`, will be automatically generated
by RethinkDB when the document is inserted.
- The field `createdAt`, when `undefined`, is automatically set to `r.now()` by thinky
and will be replaced in the database by the time at which the query is executed.

III. _Get one document by primary key_

```js
function get(request, response, next) {
  User.get(request.id).run().then(function(user) {
    res.send(JSON.stringify(user));
  }).error(function(error) {
    // Document not found, network errors etc.
    response.send(500, {error: error.message}
  });
}
```

IV. _Update a user given its primary key_

```js
function get(request, response, next) {
  User.get(request.id).update(request.body).run().then(function(user) {
    res.send(JSON.stringify(user));
  }).error(function(error) {
    // Document not found, not valid document, network errors etc.
    response.send(500, {error: error.message}
  });
}
```

If you look at this snippet, a unique query is executed, not two. ORMs usually
require you to write something like below, which executes two queries.

```js
// Works with thinky, but you do not have to run two queries.
function get(request, response, next) {
  User.get(request.id).run().then(function(user) {
    user.merge(request.body);
    return user.save()
  }).then(function(user) {
    return JSON.stringify(user)
  }).error(function(error) {
    response.send(500, {error: error.message}
  });
}
```

So what does thinky do in the first snippet?

- It first validate all the fields passed in `update`.
- It run the update query in RethinkDB.
- It validates the whole new document (returned by the update query).

Thinky validates the whole document again because it can also validation accross multiple fields
(like check that a user is more than 21 if he lives in the US, else check
that the user is more than 18).

In the most common case, you just validate the type of each field, so the third step
will never fails. If it does the document will be reverted (and only in this case
two queries are executed).


_Note_: The `user` may be returned as `undefined` if the update is a no-op query. This
is currently a regression with 2.0 (see [rethinkdb/rethinkdb#4068](https://github.com/rethinkdb/rethinkdb/issues/4068)
to track progress).


V. _Delete a user given its primary key_

```js
function get(request, response, next) {
  User.get(request.id).delete().execute().then(function(result) {
    res.send(JSON.stringify({status: "ok"}));
  }).error(function(error) {
    // Document not found, network error etc.
    response.send(500, {error: error.message}
  });
}
```

We use `execute` here and not `run` because no document will be returned.

VI. _Return all users_

```js
function all(request, response, next) {
  User.run().then(function(users) {
    res.send(JSON.stringify(users));
  }).error(function(error) {
    // Network errors etc.
    response.send(500, {error: error.message}
  });
}
```

VII. _Pagination_

```js
var perPage = 50;

function range(request, response, next) {
  var start = (request.start) ? request.start: r.minval;
  User.between(start, r.maxcal).limit(perPage).run().then(function(users) {
    res.send(JSON.stringify(users));
  }).error(function(error) {
    response.send(500, {error: error.message}
  });
}
```

Pagination here is done via primary key with `between`/`limit`,
and not with `skip`/`limit` for performance reasons.


Need the number of users?

```js
function range(request, response, next) {
  User.count().execute().then(function(count) {
    res.send(JSON.stringify(count));
  }).error(function(error) {
    response.send(500, {error: error.message}
  });
}
```

This is it! Stay tuned for the next article!
