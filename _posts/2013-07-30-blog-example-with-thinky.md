---
layout: post
category : Geek 
tags : [thinky, rethinkdb, orm, javascript, express, blog]
title: Blog example with Thinky
---
{% include JB/setup %}

I have recently been working on [thinky](https://github.com/neumino/thinky) (a JavaScript ORM for RethinkDB), and I just finished adding a new example on how to use thinky.

- [Demo](http://thinky-blog.justonepixel.com)
- [Code source](https://github.com/neumino/thinky/tree/master/examples/blog)

This example is a blog built with Node, Thinky, Express and AngularJS. The purpose of this example is to illustrate how to use Thinky so I will not spend time writing about AngularJS or Express. If people are interested, I wouldn't mind writing about it, but I added some comments in the code, so understanding how the stack works shouldn't be too hard (especially since these frameworks are quite user-friendly).


There are two files that use Thinky:

- [thinky.js](https://github.com/neumino/thinky/blob/master/examples/blog/thinky.js)
- [routes/api.js](https://github.com/neumino/thinky/blob/master/examples/blog/routes/api.js)


### <a href="https://github.com/neumino/thinky/blob/master/examples/blog/thinky.js">thinky.js</a>

In this file, we are just loading the module with `require()` and call `.init()` which will create a pool of connections that will be used to make queries.

```javascript
thinky.init({
    host: config.host,
    port: config.port,
    db: config.db
});
```

### <a href="https://github.com/neumino/thinky/blob/master/examples/blog/routes/api.js">routes/api.js</a>

This file contains all the interesting things about thinky.

<strong>Models</strong>

We first create models with `thinky,createModel()`

```javascript
var Post = thinky.createModel('Post', {
    id: String,
    title: String,
    text: String,
    authorId: String,
    date: {_type: Number, default: function() { return Date.now() }}
});
var Author = thinky.createModel('Author', {
    id: String,
    name: String,
    email: String,
    website: String
});
var Comment = thinky.createModel('Comment', {
    id: String,
    name: String,
    comment: String,
    postId: String,
    date: {_type: Number, default: function() { return Date.now() }}
});
```

- The first argument is the name of the model (which is also the name of the table).</li>
- The second argument is the schema. A schema is just an object where fields map to a type (`String`, `Number`, `Boolean` etc...) or to an object with a `_type` field. You can also pass options in the latter case like a default value.</li>

One really nice thing about RethinkDB is that even if it is a NoSQL database, it lets you do efficient JOINs between table. In our case, we want to create two relations:

- One post has one author. This join is performed on the condition
`post.authorId == author.id`. We would like to store the author in a field named `author` so the syntax will be:</li>

```javascript
Post.hasOne( Author, 'author', {leftKey: 'authorId', rightKey: 'id'})
```

- One post can have multiple comments. This join is performed on the condition
`post.id == comment.postId`. We would like to store the joined comments in the field `comments` so the syntax will be:

```javascript
Post.hasMany( Comment, 'comments', {leftKey: 'id', rightKey: 'postId'}, {orderBy: 'date'})
```

The last object with the field `orderBy` are options of the JOIN operation. In this case, we want the comments to be ordered by their date.</li>

Now that we have set all our models, let's look at how we use our models to make queries:

__Basic operations__

- To retrieve a single post, the syntax is pretty close to the ReQL one:

```javascript
Post.get(id).run(function(error_post, post) { ... })
```

In ReQL the query would be

```javascript
r.db("blog").table("Post")
    .get(id)
    .run(connection, function(error_post, post) { ... })
```

The main difference here is that you do not need to deal with connection when using Thinky. Thinky will take care of maintaining a pool of connections.</li>

- Like in ReQL, you can chain commands with Thinky. While chaining is really nice, you may sometimes want to be able to execute a query without using `.run()`. In this case you can just pass an extra callback to any function. For example, this query

```javascript
Author.get(id, function(error, author) { ... })
```
is the same as

```javascript
Author.get(id).run(function(error, author) { ... })
```

- Saving an object in the database is as simple as calling `.save()` on an instance of a model.

```javascript
var newPost = new Post(req.body);
newPost.save(function(error, result) { .. })
```
The ReQL query being:

```javascript
r.db("blog").table("Post").insert(req.body)
    .run(connection, function(error, result) { ...})
```

- In a similar way, you can update an object by calling `.update()`.

```
var newPost = new Post(req.body);
newPost.update( function(error, post) { ... })
```

The equivalent ReQL query would be

```javascript
r.db("blog").table("Post").get(req.body["id"])
    .update(req.body)
    .run(connection, function(error, result) { ...})
```

Note: If you call `save()` twice on an object created with `new`, it will use `insert()` the first time, and `update()` the second time. The reason why Thinky does not use `upsert` by default is because I believe it could lead to undesired deletions/updates.</li>

- If you want to delete the document with a certain id, you will have to select it with `.get()` first then call `.delete()` on it.

    ```javascript
    Post.get(id).delete( function(error, result) { ... })
    ```

    Which is in ReQL:

    ```javascript
    r.db("blog").table("Post")
        .get(id)
        .delete()
        .run(connection, function(error, result) { ... })
    ```
    Once nice thing with ReQL is that the deletion we did was done in only one query. Another way to do it would be to fetch the document with <codeget()` then call `delete()` on the document, but that would fire two ReQL queries.


__Cool things__

- Let's now look at the nice things that Thinky provides. The first query you can read in the api.js file retrieves all the posts, orders them by date in a descending order and retrieves all the joined documents (that is to say the author and the comments).

    This is how you would do it with Thinky:

    ```javascript
    Post.orderBy('-date').getJoin().run(function(error, posts) { ... })
    ```

    The equivalent ReQL query is:

    ```javascript
    r.db("blog").table("Post").orderBy(r.desc("date"))
        .map( function(post) {
            return doc.merge({
                author: r.db("blog").table("Author").get(post("authorId"))
                comments: r.db("blog").table("Comment")
                .getAll( doc("id"), {index: "postId"})
                .coerceTo("array")
            })
        }).run( connection, function(error, posts) { ... } )
    ```
    Note 1: that everything happens in the database. Thinky <strong>does not</strong> process data.</li>
    Note 2: Thinky currently does not use the ReQL `eqJoin` command because it behaves like an inner join (see <a href="https://github.com/rethinkdb/rethinkdb/issues/1152">this github issue</a>)

- You can also retrieve joined documents for only one element. You just have to call `getJoin()` like before:

    ```javascript
    Post.get(id).getJoin().run(function(error, post) { ... })
    ```

The other queries in routes/api.js are similar to the previous ones but are done on other tables.


Thinky is a new library. If you have any feedback or suggestions, I would love to read them!
