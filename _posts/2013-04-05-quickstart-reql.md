---
layout: post
category : Geek
tags : [quickstart, reql, rethinkdb, javascript]
title: Quickstart with ReQL
---
{% include JB/setup %}

This (not so quick) start will rely on JavaScript. If you want to use Python or
Ruby, don't look at the parts that deal with a callback.

This article can be splitted in 4 parts

1. Connection and meta queries
2. Usual operations
3. Joins
4. Map-reduce

### Connection and meta queries

Start your server with

```bash
$ rethinkdb
```

Start node.js. The code starts with a require

```js
var r = require('rethinkdb');
```

All the functions we are going to use, are methods of the variable r.

Then connect to the server. if you haven't started the server on your local
machine or have defined used the flag --port-driver or --port-offset, update the
port value (Look for the line info: Listening for client driver connections on port)

```js
var connection = null;
r.connect( {host: 'localhost', port: 28015} , function(err, conn) {
    if (err) throw err;
    connection = conn;
}
```

The callback defined in the JavaScript driver use the node convention `function(err, result) {}`

This is how you list the databases you have

```js
r.dbList().run( connection, function(err, result) {
    if (err) throw err;
    util.log(result);
})
```

`r.dbList()` is the query that is going to be built by the JavaScript driver.
When you are going to call .run() on it, the query is going to be compiled in
a binary array (using google protobuf library) then send to the server. When the
driver get the response back, it executes the callback.

If you just started RethinkDB, you should see a database test.

Now let's create a database blog.

```js
r.dbCreate("blog").run( connection, function(err, result) { ... })
```

You can check that it was created with .dbList()
Now create a table users.

```js
r.db("blog").tableCreate("users")
    .run( connection, function(err, result) { ... } )
```

The query starts with r, then we "select" the database blog and then we create a
table users. If you log the results, you should see { created: 1}

You can verify again that the table was created with .tableList()

```js
r.db("blog").tableList()
    .run( connection, function(err, result) { ... } )
```

Again, we select the database blog first and only then list the tables.

### Usual operations

Let's insert some data now in the database users

```js
r.db("blog").table("users")
    .insert( {
        name: "Michel",
        age: 26,
        karma: 108,
        gender: "male"
    }).run( connection, function(err, result) { ... } )
```

The callback should log an object like this

```js
{
    "inserted": 1,
    "generated_keys":[
        "3c05a2cc-de49-463c-9a7e-abeeef7f9568"
    ]
}
```

When we created a table with .tableCreate(), we just passed the name of the table,
so RethinkDB pick the default name for the primary key which is id.

The object we inserted didn't have a value for id, so RethinkDB just generated
a random value for us `3c05a2cc-de49-463c-9a7e-abeeef7f9568`.

Let's retrieve our data.

```js
r.db("blog").table("users").run( connection, function(err, cursor) {
    cursor.toArray( function(err, result) {
        util.log(result);
    })
})
```

Wait, what is this double nested callbacks?

`r.db("blog").table("users")` is going to return all the documents in the table
users with a stream (that can be huge). That's why the callback in run() will
provide a cursor.

Here we call `toArray( callback )` that is going to convert the cursor to an array
and call our callback with it. A RethinkDB cursor comes with other useful methods
like .each(), hasNext() and next(). There is an [issue on github](https://github.com/rethinkdb/rethinkdb/issues/604) to make the API
more consistent.

Let's insert a little more data before going further.

```js
r.db("blog").table("users")
    .insert([
        {name: "Laurent", age: 28, karma: 42, gender: "male" },
        {name: "Sophie", age: 22, karma: 57, gender; "female" },
        {name: "Aurelie", age: 18, karma: 90, gender: "female" },
        {name: "Dan", age: 15, karma: 31, gender: "mal" },
    ]).run( connection, function(err, result) { ... } )
```

To extract all users that are more than 21, we can use a filter that way:

```js
r.db("blog").table("users")
    .filter(r.row("age").ge(21))
    .run( connection, function(err, cursor) { ... } )
```

The method filter() take a predicate as argument. Here, `r.row` select the current
row being accessed while `("age")` will select the attribute age. In Python or Ruby,
the equivalent syntax is r.row["age"]. JavaScript doesn't allow to overwrite the
square brackets operator, so it has been replaced with parentheses. Once we have
the age of the user, we compare it with ge() which stands for greater or equal.

That is one way to do thing. A real cool thing about RethinkDB is that you can pass
anonymous functions (also known as lambda functions). Here is the same query with
an anonymous function.

```js
r.db("blog").table("users")
    .filter( function(user) {
        return user("age").ge(21)
    }).run( connection, function(err, cursor) { ... } )
```

If you want to update all the users that are less than 21 with a boolean value 
`{ can_drink: false}`, you have first to select your data with a filter() and
then to update your data with update(). Here is how it is done

```js
r.db("blog").table("users")
    .filter(r.row("age").lt(21))
    .update( { can_drink: false } )
    .run( connection, function(err, result) { ... } )
```

RethinkDB provides a way to completely replace a document with the method replace. That's also how you remove an attribute. Let's remove the attribute can_drink from our users who are more than 18. The philosophy stay the same. We first select our users, then replace them.

```
r.db("blog").table("users")
    .filter(r.row("age").ge(18))
    .replace(r.row.without("can_drink"))
    .run( connection, function(err, result) { ... } )
```

`r.row.without("can_drink")` returns the current element without the field can_drink.

If you want to delete all users less than 18, you have to go with the command delete().

```
r.db("blog").table("users")
    .filter(r.row("age").lt(18))
    .delete()
    .run( connection, function(err, result) { ... } )
```

One other way to select a result is using get(). The example below is going to return one document whose primary key is "3c05a2cc-de49-463c-9a7e-abeeef7f9568". This query is more efficient than a simple filter because it will do the search with a B-tree. Right now, get() only supports primary key. It will soon be able to do the same with a secondary index (See [#602](https://github.com/rethinkdb/rethinkdb/issues/602) for progress).

```
r.db("blog").table("users")
    .get("3c05a2cc-de49-463c-9a7e-abeeef7f9568")
    .run( connection, function(err, result) { ... } )
```

That's all for the common operations.

### Joins

Let's take a look at how to do joins. Even if RethinkDB is a NoSQL database, it
provides efficient joins with the .eqJoin() method.

Let's suppose that we have a table comments with the following schema

```js
{
    id: <string>
    id_author: <string>
    comment: <string>
}
```

To retrieve all the comments with the name of the user, we can do a join this way:

```js
r.db("blog").table('comments').eqJoin(
    "id_author",
    r.db("blog").table('users')
    ).zip()
.run( connection, function(err, result) { ... } )
```

We are going to take a stream of comments, and for each comment, we are going to
look for users whose id match id_author of the current comment.

`zip()` is a going to merge the comment and the author in the same document.
If you want to do a join on non-primary keys, you can use `innerJoin()` or `outerJoin()`

### Map reduce

Let's now look at the cool things. RethinkDB provides an efficient way to do map/reduce.
Suppose you want to know what is the sum of karma

```js
r.db("blog").table("users")
    .map(r.row("karma"))
    .reduce(function(a, b) {
        return a.add(b)
    }, 0)
    .run( connection, function(err, result) { ... } )
```

The method map is going to map the value of the field karma, then reduce is going to make the sum of all the value of karma.

The second parameter of reduce() is the base. A naive implementation would look like that

```js
map    -> [108, 42, 57, 90 ]
reduce -> 0 + 108 + 42 + 57 + 90 -> 297
```

The method reduce() is implemented in parallel, so you are not free to specify whatever you want as a base. Suppose that the first two documents are on a server and the last two are on a second server.

What happens is going to be something like that

```
map    -> [    108, 42,           57, 90      ]
           data on server 1 - data on server 2

reduce ->  Server 1 
           0+108 ->1108
           108+42 -> 150
           -------------
           Server 2
           0+57 -> 57
           57+90  -> 147
           -------------
           150+147 -> 297
```

So if you want to retrieve the sum of the karma plus 10, you can not pass 10 as
a base. You have to add 10 after the reduce. The base has to be neutral.

RethinkDB also provide a groupedMapReduce method that is going to group rows,
do a map, and eventually reduce the results.

Here is how to compute the sum of the karma of the two genders.

```js
r.db("blog").table("users").groupedMapReduce(
    r.row("gender"), // group by gender
    r.row("karma"), // map the value of karma
    function(a, b) { return a.add(b) }, // sum the karma
    0
).run( connection, function(err, result) { ... } )
```

And that's all for the quick start. This was a quick glance at how to do common operations and some little fancy (yet useful) things like join and map-reduce. I didn't talk about everything, like how to use a default database etc. If you want to know more about Rql, take a look at the [API docs](http://rethinkdb.com/api).

