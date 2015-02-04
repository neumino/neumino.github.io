---
layout: post
category : Geek
tags : [rethinkdb, rethinkdbdash, thinky, extend, orm]
title: "Replicate ReQL API in your own classes"
---
{% include JB/setup %}


ReQL is embedded in the host language. So if you use JavaScript,
you do not have to concatenate (and escape) SQL strings, or build JSON
objects with some special keys. Your queries are plain JavaScript.


For example, using `rethinkdbdash`, updating all the users that are
at least 18 years old with a field `isAdult` set to `true` can be written
like this:

```js
var promise = r.table('users').filter(function(user) {
  return user('age').gt(18)
}).update({ isAdult: true }).run();
```

RethinkDB ships with a data explorer where you can get test your queries.
Because ReQL is composed of chainable commands, the data explorer can provide
you with suggestions and auto-completion, making learning ReQL easy and fun.

### Replicate the API

But this is not the only nice part in having an embedded chainable language;
you can reproduce the query language in your own classes by just copying
the commands. This is basically what [thinky](https://thinky.io) (a Node.js ORM)
is doing and this has three benefits:

- It is easy to do; it is way easier than implementing your own methods to query the database.
- Because the API is the same, the learning curve is a [Heaviside step](http://en.wikipedia.org/wiki/Heaviside_step_function).
- You can extend ReQL with your own methods and aliases.

This article aims to explain how to replicate ReQL API using rethinkdbdash. The same
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

You can import all the methods with the following code.

```js
var r = require('rethinkdbdash')();
var Term = r.expr(1).__proto__;

function Query(query) {
  this._query = query; // an instance of Term
};

(function() {
  for(var key in Term) {
    Query.prototype[key] = function() {
      return new Query(this._query[key].apply(this._query, arguments));
    }
    User.prototype[key] = function() {
      var table = r.table('users');
      return new Query(table[key].apply(table, arguments));
    }
  }
})();
```

Basically we are doing three things:

- We create a simple wrapper `Query` around a rethinkdbdash query (an instance of `Term`).
- We define all the methods from `Term` in `Query.prototype` that create a new `Query` with the new `Term`
- We define all the methods from `Term` to `User.prototype` that will return a `Query` with the query `r.table('users')`.

Now you can do:

```js
// Set the name of the user with id 1
var promise = Users.get(1)
  .update({name: "Michel"})
  .run()

// Set all the users as "not verified"
var promise = Users.update({verified: false}).run();

// Perform a simple join to retrieve the emergency contact person
var promise = Users.get(1).merge(function(user) {
  return { ecpPerson: Users.get(user.ecp) }
}).run();
```

So if you feel like thinky is too cumbersome for your project (like if you do
not need relations, hooks etc.) you can easily replicate the same API.

### Define your own methods

You can also define your own methods this way:

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
var promise = Users.isAdult().run();
```

You can also overwrite `run` to return instance of the class `User` instead of normal objects.


### Notes

- If you do not plan to create your own methods, you do not need to create `Query`, and you can
just return ReQL queries.

```js
var r = require('rethinkdbdash')();
var Term = r.expr(1).__proto__;

(function() {
  for(var key in Term) {
    User.prototype[key] = function() {
      return r.table('users')[key].apply(table, arguments));
    }
  }
})();
```


- The reason why we create a new instance of `Query` and not mutate the
`Query` object is to let people "fork" queries, like:

```js
var Adults = Users.filter(function(user) { return user("age").gt(18) });
Adults.update({isAdult: true}).run().then(...).error(...);
Adults.filter({location: "US"}).run().then(...).error(...);
```

- The immediately-invoked function is not optional.
