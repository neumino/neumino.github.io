---
layout: post
category : Geek
tags : [rethinkdb, rethinkdbdash, thinky, extend, orm]
title: "A hidden gem in ReQL"
---
{% include JB/setup %}


ReQL is embedded in the host language, so if you use JavaScript for example,
you do not have to concatenate and escape some SQL strings, or build some JSON
objects with a few special keys. Your queries are plain JavaScript.


For example, using `rethinkdbdash`, you can update all the users that are
at least 18 years old with a field `isAdult` set to `true`. One way to write
the query is:

```js
var promise = r.table('users').filter(function(user) {
  return user('age').gt(18)
}).update({
  return { isAdult: true }
}).run();
```

RethinkDB ships with a data explorer where you can get test your queries.
Because ReQL is composed of chainable commands, the data explorer can provide
you with suggestions and auto-completion.

But this is not the only nice part in having an embedded chainable language;
you can reproduce the query language in your own classes by just copying
the commands. This is basically what [thinky](https://thinky.io) (a Node.js ORM)
is doing.

This article aims to explain how to perform such thing with rethinkdbdash. The same
result can be done with the official driver. The only thing to know about
rethinkdbdash, is that all commands return an instance of the class `Term`, and that
`Term` implements all the methods like `filter`, `get`, `update` etc.

Suppose that you have a class `User`.

```js
function User(data) {
  this.id = data.id;
  this.name = data.name;
  this.email = data.email;
  this.ecp = data.ecp || null; // id of the emergency contact person
}
```

You can import all the methods with the following code.

```js
var r = require('rethinkdbdash')();
var Term = r.expr(1).__proto__;

function Query(query) {
  this._query = query; // an instance of Term
}

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

Basically what we are doing three things:

- We create a simple wrapper `Query` around a rethinkdbdash query (an instance of `Term`)
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

// Perform a simple join retrieve the emergency contact person
var promise = Users.get(1).merge(function(user) {
  return { ecpPerson: Users.get(user.ecp) }
}).run();
```

So if you feel like thinky is too cumbersome for your project (like if you do
not need relations, hooks etc.) you can easily replicate the same API.



_Note 1_: The reason why we create a new instance of `Query` and not mutate the
`Query` object is to let people "fork" queries, like:

```js
var Adults = Users.filter(function(user) { return user("age").gt(18) });
Adults.update({isAdult: true}).run().then(...).error(...);
Adults.filter({location: "US"}).run().then(...).error(...);
```

_Note 2_: The anonymous function that get immediately called is not optional.
