---
layout: post
category : Geek
tags : [thinky, relation, rethinkdb]
title: "Rethinkdbdash and client side backtraces"
---
{% include JB/setup %}

### A bit of context

I released rethinkdbdash 2.1.3 (and 2.1.4) a bit earlier today. This release besides adding
a few missing syntaxes for 2.1 (`r.union` and `condition.branch(...)`),
fixed an issue with the client-side backtraces, though to be honest, the client-side
backtraces were not really working before.

If a RethinkDB query throws an error, the server will provide a backtrace and the
driver will reconstruct the query and underline the broken part. For example
if a table is missing you will see this error:

```js
ReqlOpFailedError: Table `test.users` does not exist in:
r.db("test").table("users").filter(function(var_1) {
^^^^^^^^^^^^^^^^^^^^^^^^^^^                         
    return var_1("age").gt(18)
})
```

The query is printed back and the broken parts are highlighted. These backtraces are in my
opinion of the little one of the little things that makes RethinkDB enjoyable to use.
However a few queries cannot be sent to the server and therefore no backtraces are available.
Because RethinkDB stores JSON document, some JavaScript values are forbidden, typically `NaN` and `Infinity`.
When the driver finds a forbidden value, it used to just throw an error saying `NaN cannot be converted to JSON`.
This is painful for two reasons:

- The presence of `NaN` is an operational error and the driver should just reject the query, not throw.
- `NaN` can easily propages and you basically have to inspect all possible variables. The bigger your
query, the harder debugging it is.


### What's new

Since 2.1.3, Rethinkdbdash now builds backtraces for these errors and
will point exactly where the problem is.


```js
var r = require('rethinkdbdash')();

var ADULT_AGE = 18;
var ADULT_AGE_US = NaN; // oops

r.db('test').table('users').merge(function(user) {
  return r.branch(
    user('location').eq('US'),
    { canDrink: user('age').gt(ADULT_AGE_US) },
    { canDrink: user('age').gt(ADULT_AGE) }
  )
}).run().then(console.log).error(function(error) {
  console.log(error);
});
```

What will be printed is:

```js
ReqlRuntimeError: Cannot convert `NaN` to JSON in:
r.db("test").table("users").merge(function(var_1) {
    return r.branch(var_1("location").eq("US"), {
        canDrink: var_1("age").gt(NaN)
                                  ^^^ 
    }, {
        canDrink: var_1("age").gt(18)
    })
})
```

Pretty cool uh? You can pin exactly where the `NaN` value is.


Feedback/suggestions? Ping me on Twitter via [@neumino](https://twitter.com/neumino)
or shoot me an email at [orphee@gmail.com](mailto:orphee@gmai.com).
