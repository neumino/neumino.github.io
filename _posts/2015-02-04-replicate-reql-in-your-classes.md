---
layout: post
category : Geek
tags : [rethinkdb, rethinkdbdash, thinky, extend, orm]
title: "Replicate ReQL API in your own classes"
---
{% include JB/setup %}


ReQL is embedded in the host language. If you use JavaScript,
you do not have to concatenate and escape SQL strings, or build JSON
objects with some special keys; your queries are plain JavaScript.


For example with [rethinkdbdash](https://github.com/neumino/rethinkdbdash),
updating all the users that are at least 18 years old with a field `isAdult`
set to `true` (in one round trip), can be written like this:

```js
var promise = r.table('users').filter(function(user) {
  return user('age').gt(18)
}).update({ isAdult: true }).run();
```

RethinkDB ships with a data explorer where you can get test your queries.
Because ReQL is composed of chainable commands, the data explorer can provide
you with suggestions and auto-completion, making learning ReQL easy and fun.

### Replicate the API

But this is not the only advantage of having an embedded chainable language;
you can import the query language in your own classes by just copying
the commands. This is what [thinky](https://www.justonepixel.com/thinky) (a Node.js ORM)
does and this has multiple benefits:

- It is easy to do and require very little work.
- Because the API is the same, the learning curve is a [Heaviside step](http://en.wikipedia.org/wiki/Heaviside_step_function).

This article explains how to replicate ReQL API using rethinkdbdash. The same
operation can be done with the official driver or in another _flexible_ language like Python, Ruby etc.
The only thing to know about rethinkdbdash, is that all commands return an instance of the class `Term`, and that
`Term` implements all the methods like `filter`, `get`, `update` etc.

Suppose that you have a class `User`.

```js
function User(data) {
  this.id = data.id;
  this.name = data.name;
  this.email = data.email;
  this.ecp = data.ecp || null; // id of the emergency contact person
};
```

You can import all the methods in `User` by looping over all the keys
in `Term`.

```js
var r = require('rethinkdbdash')();
var Term = r.expr(1).__proto__;

for(var key in Term) {
  (function(key) { // this immediately invoked function is required
    User.prototype[key] = function() {
      return r.table('users')[key].apply(table, arguments);
    }
  })(key)l
}
```

Now you can write:

```js
// Set the name of the user with id 1
Users.get(1).update({name: "Michel"}).run().then(...).error(...);

// Set all the users as "not verified"
Users.update({verified: false}).run().then(...).error(...);

// Perform a simple join to retrieve the emergency contact person
Users.get(1).merge(function(user) {
  return { ecpPerson: Users.get(user.ecp) }
}).run().then(...).error(...);
```

So if thinky is too cumbersome for your project (if you do
not need relations, hooks etc.) you can easily replicate the same API.


### Define your own ReQL methods

You can also define methods on `Users` by simply adding them in `Users.prototype`, but
you can also create your own methods on the queries by wrapping the ReQL queries:

```js
var r = require('rethinkdbdash')();
var Term = r.expr(1).__proto__;

function Query(query) {
  this._query = query; // an instance of Term
};

for(var key in Term) {
  (function(key) {
    if (key === "run") {
      Query.prototype[key] = function() {
        // We want to return the promise returned by the ReQL query here
        return this._query.run.apply(this._query, arguments));
      }
    } else {
      Query.prototype[key] = function() {
        return new Query(this._query[key].apply(this._query, arguments));
      }
    }

    User.prototype[key] = function() {
      return r.table('users')[key].apply(table, arguments)
    }
  })(key);
}
```

A few notes about this code:

- You can copy all the methods on `Query.prototype` except `run` since for this
method you want to return the promise itself.
- Each method returns a new `Query` object to enable forking:

```js
var Adults = Users.filter(function(user) { return user("age").gt(18) });
Adults.update({isAdult: true}).run().then(...).error(...);
Adults.filter({location: "US"}).run().then(...).error(...);
```

This is it! You can now define your own methods:

```js
Query.prototype.isAdult = function() {
  return new Query(this._query.filter(function(user) {
    return user("age").gt(18)
  }));
};
User.prototype.isAdult = function() {
  var table = r.table('users');
  return new Query(table).isAdult();
};

// Retrieve all the adults
var promise = Users.filter({location: "UK"}).isAdult().run();
```

Questions? Feedback? Shoot me a mail at [orphee@gmail.com](mailto:orphee@gmail.com)
or ping me on Twitter [@neumino](https://twitter.com/neumino)
