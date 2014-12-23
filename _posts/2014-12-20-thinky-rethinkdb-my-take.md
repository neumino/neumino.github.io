---
layout: post
category : Geek
tags : [thinky, rethinkdb, orm, nodejs]
title: "Thinky and its Raison d'être!"
---
{% include JB/setup %}

*Disclaimer: This post reflects my opinion and just mine. I also used to work at RethinkDB.*

I am the author of [thinky](https://thinky.io). For those not familiar with it,
thinky is a [Node.js](https://nodejs.org) ORM for [ReahinkDB](http://rethinkdb.com). I never really
took the time to write about its philosophy and its raison d'être. Since I
have some time during these holidays, here we are.

### A bit about RethinkDB

NoSQL databases come in multiple flavors, and describing all of them is out of
this article's scope. However, talking about thinky's philosophy without
describing RethinkDB would be pretty hard. RethinkDB wrote an interesting
[blog post](http://www.rethinkdb.com/blog/mongodb-biased-comparison/) about
where it stands. My thoughts on the subject are similar but slightly different:
RethinkDB is development friendly.

- RethinkDB is schemaless. 
  - Flexible schemas are like dynamically typed languages. They
are easier to learn, faster to bootstrap with.  
  - No schema means no migration to update a schema.

- [ReQL](http://rethinkdb.com/docs/introduction-to-reql/) (RethinkDB Query
Language) is embedded in the host language, meaning no more SQL strings or
[MongoDB](https://www.mongodb.org/) JSON objects to build. In JavaScript, you
end up with a chainable query language:

    ```js
    var promise = r.table("users").get("67dc69ae-e235-4f55-a71b-6b87fe4df894")
        .update({name: "Michel"}).run(connection);
    ```

- RethinkDB has efficient/distributed server-side joins. Nested structures is a [poor answer](http://www.sarahmei.com/blog/2013/11/11/why-you-should-never-use-mongodb/)
for many-to-many relations, which appear as soon as you try to
model a social network with users having friends.

- Setting up shards and replicas is a matter of a few clicks on a gorgeous and
friendly web interface that is natively shipped with the server.

- RethinkDB provide an easy way to push changes to clients. Broadcasting all the
changes on the table `data` is as simple as:

    ```js
    var sockets = []; // All your SockJS connections
    r.table("data").changes().run(connection).then(function(feed) {
      feed.each(function(change) {
        var message = JSON.stringify(change.new_val);
        for(var i=0; i<sockets.length; i++) {
          sockets[i].write(message);
        }
      })
    })
    ```

    [Meteor](https://www.meteor.com/) and [Firebase](https://www.firebase.com/) are
    hooked on MongoDB operation logs, Asana built their famous [Luna](https://asana.com/luna)
    framework on top of [Kraken](https://github.com/Asana/kraken), their
    distributed pubsub server. Implementing such systems is hard and complicated
    (kudos to all these engineers). RethinkDB provide this real-time feature with
    no cost, and without locking you to a whole stack.


### What does thinky do?

Thinky works in harmony with RethinkDB to provide the developer a frictionless
experience. This is done by automating/reducing the work required for common
operations.

- Validate your data before saving it. Flexible schemas mean faster
iterations but corrupted data is the worse nightmare for any engineer,
so thinky does all the work to make sure that you only save valid documents. It
can also easily generate default values for you.

- Handle connections under the hood in an optimal way. No need for middleware
to open/close connections, no need for listeners to handle network errors.

    ```js
    Users.get("67dc69ae-e235-4f55-a71b-6b87fe4df894").run().then(function(user) {
      // ...
    }).error(...)
    ```

- Define the relations once and automatically save/retrieve joined documents with
a simple command `getJoin`.

    ```js
    var Post = thinky.createModel("Post", { id: String, title: String, content: String, idAuthor: String }); 
    var Author = thinky.createModel("Author", { id: String, name: String });
    Post.belongsTo(Author, "author", "idAuthor", "id");
    //                       |-> key where the joined document will be stored
    //                                  |-> left key
    //                                            |-> right key

    Post.get("67dc69ae-e235-4f55-a71b-6b87fe4df894").getJoin().run().then(function(post) {
      // post will have a field "author" that maps to its author.
    }).error(...);
    ```

- Extend ReQL which results in an incredibly powerful query language with
very little to learn. Anonymous functions that get [serialized](http://rethinkdb.com/blog/lambda-functions/) and sent to the server
are still available as inner queries are.

    ```js
    // Return the grown up friends'id of a user.
    User.get("67dc69ae-e235-4f55-a71b-6b87fe4df894").do(function(user) {
      return user("friend_ids").filter(function(friend_id) {
        return Users.get("friend_id")("age").gt(18);
      });
    }).execute().then(function(friend_ids) {
      // ...
    }).error(...)
    ```

- Automatically create tables for you. Spend times on things that matter, your code,
not operations.


### What does thinky not do?

Thinky is also built as a genuine useful library. It does not try to do
what it cannot, or give the user the illusion of a feature when it is
not safe.

- Thinky does not provide unique secondary indexes as RethinkDB does not (like to the extent
of my knowledge, any databases in a distributed scenario).
- Thinky does not provide transactions as RethinkDB provides atomicity [per document](http://rethinkdb.com/docs/architecture/#how-does-the-atomicity-model-work)
only.


### Conclusion

**Should you use RethinkDB?**   
There are a few use cases where you are better off with another database.

- RethinkDB does not support transactions. If you need strong consistency (like if you
are building a bank system), use a SQL database with ACID properties like [PostGreSQL](http://www.postgresql.org/)
(or maybe [FoundationDB](https://foundationdb.org) though it is not open source).
- Flexible schemas come with a price. Documents are not stored like it is in
column oriented database. In some cases, you may be better off with something
like [HBase](http://hbase.apache.org/).
- Large clusters for RethinkDB are not [properly scaling](http://rethinkdb.com/stability/),
though this seems to be [fixed](https://github.com/rethinkdb/rethinkdb/issues/3198)
and should be released in the next version. Small clusters are pretty stable and
the majority of you probably do not need more.
- If you need only a key-value store, [Cassandra](http://cassandra.apache.org/)
is a pretty good one in my opinion.

That being said, if you build a common web application (like Yelp, Feedly etc.),
you would probably have a lot to gain from using it.

**Should you use thinky?**  
If you use RethinkDB and Node.js, yes. Thinky works in harmony with RethinkDB to make writing code easier and
faster by doing common things like validation. It is also a simple:

- There is little to learn if you know ReQL, the syntaxes are almost [the same](https://github.com/neumino/thinky/blob/3b4aa9d0fc120c5d99b438328204a3acfa799d1a/lib/query.js#L375).

While being simple, it is also a powerful library:

- Custom validations let you leverage all the power
of libraries like [validator](https://github.com/chriso/validator.js).
- [Hooks](http://thinky.io/documentation/api/model/#pre) let you use powerful
API like [Mailgun's API](http://documentation.mailgun.com/api-email-validation.html#email-validation)
for email verification.

So give it a shot, and if you have feedback/suggestions, open an issue on [GitHub](https://github.com/neumino/thinky/issues/new),
ping me on Twitter via [@neumino](https://twitter.com/neumino),
or shoot me a mail at [orphee@gmail.com](mailto:orphee@gmai.com).
