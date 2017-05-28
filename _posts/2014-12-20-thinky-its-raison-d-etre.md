---
layout: post
category : Geek
tags : [thinky, rethinkdb, orm, nodejs]
title: "Thinky and its Raison d'être!"
---
{% include JB/setup %}

*Disclaimer: This post reflects my opinion and just mine. I also used to work at RethinkDB.*

I am the author of [thinky](https://www.justonepixel.com/thinky). For those not familiar with it,
thinky is a [Node.js](https://nodejs.org) ORM for [RethinkDB](http://rethinkdb.com).
I never really took the time to write about the philosophy behind thinky. Since I
have some time during these holidays, here we are.

### A bit about RethinkDB

NoSQL databases come in multiple flavors, and describing all of them is out of
this article's scope. However, talking about thinky's philosophy without
describing RethinkDB would be pretty hard. RethinkDB wrote an interesting
[blog post](http://www.rethinkdb.com/blog/mongodb-biased-comparison/) about
where it stands. While I agree with their post, I think they don’t place
enough emphasis on how developer friendly RethinkDB is.

- RethinkDB is schemaless. A schemaless database is like a dynamic programming
language; they are both easier to learn and allow for faster bootstrapping.
Additionally, changing the format of your data doesn’t require any migration.

- [ReQL](http://rethinkdb.com/docs/introduction-to-reql/) (RethinkDB Query
Language) is embedded in the host language, meaning no more SQL strings or
[MongoDB](https://www.mongodb.org/) JSON objects to build. In JavaScript, you
end up with a chainable query language:

    ```js
    var promise = r.table("users").get("67dc69ae-e235-4f55-a71b-6b87fe4df894")
        .update({name: "Michel"}).run(connection);
    ```

- RethinkDB has efficient and distributed server-side joins. Nested structures
are a [poor answer](http://www.sarahmei.com/blog/2013/11/11/why-you-should-never-use-mongodb/)
for many-to-many relations, a situation that appears as soon as you try to
model a social network with users having friends.

- RethinkDB can set up shards and replicas with just a few clicks on a gorgeous and
friendly web interface that natively ships with the server.

- RethinkDB provides an easy way to push changes to clients -- broadcasting all the
changes on the table `data` is as simple as this:

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

    While [Meteor](https://www.meteor.com/) and [Firebase](https://www.firebase.com/)
    are hooked on MongoDB operation logs and [Asana](https://asana.com) built
    their famous [Luna](https://asana.com/luna) Framework on top of [Kraken](https://github.com/Asana/kraken)
    (their distributed pubsub server), it is hard and complicated to build such
    systems. RethinkDB provides this real-time feature at no additional cost,
    without locking you to a whole stack.

### What does thinky do?

Thinky works in harmony with RethinkDB to provide a frictionless experience
for the developer. This is done by automating and reducing the work required
for common operations.


- Thinky validates your data before saving it. Flexible schemas mean faster
iterations but corrupted data is any engineer’s worst nightmare; thinky does
all the work to make sure that you only save valid documents. It can also
easily generate default values for you.

- Thinky handles connections under the hood in an optimal way. There’s no need
for middleware to open/close connections, and no need for listeners to handle
network errors

    ```js
    Users.get("67dc69ae-e235-4f55-a71b-6b87fe4df894").run().then(function(user) {
      // do something with `user`
    }).error(...)
    ```

- Define the relations once and automatically save/retrieve joined documents with
the simple command, `getJoin`.

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

- Thinky encompasses all of ReQL’s powerful features. Anonymous functions that
get [serialized](http://rethinkdb.com/blog/lambda-functions/) and sent to the
server are still available, as are inner queries.

    ```js
    // Return the grown up friends'id of a user.
    User.get("67dc69ae-e235-4f55-a71b-6b87fe4df894")("friend_ids")
        .filter(function(friend_id) {
          return Users.get("friend_id")("age").gt(18);
        });
    }).execute().then(function(friend_ids) {
      // ...
    }).error(...)
    ```

- Thinky automatically creates tables for you. Spend time on things that matter, your code,
not operations.


### What does thinky not do?

Thinky is built as a genuine useful library. It does not try to do
what it cannot, or give the user the illusion of a feature when it is
not safe.

- Thinky does not provide unique secondary indexes. To the extent of my
knowledge, no databases support such a feature in a distributed scenario. 

- Thinky does not provide transactions as RethinkDB only provides atomicity
[per document](http://rethinkdb.com/docs/architecture/#how-does-the-atomicity-model-work).


### Conclusion

**Should you use RethinkDB?**   
There are a few use cases where you are better off with another database.

- RethinkDB does not support transactions. If you need strong consistency (i.e you
are building a bank system), use a SQL database with ACID properties like [PostGreSQL](http://www.postgresql.org/)
(or maybe [FoundationDB](https://foundationdb.org) though it is not open source).
- Flexible schemas come with a price. Documents are not stored as they are in a
column oriented database. In some cases, you may be better off with something
like [HBase](http://hbase.apache.org/).
- Operations on large clusters for RethinkDB do not [properly scale](http://rethinkdb.com/stability/);
this seems to be [fixed](https://github.com/rethinkdb/rethinkdb/issues/3198)
and should be released in the next version. Small clusters are pretty stable and
the majority of you probably do not bigger clusters.
- If you only need a key-value store, in my opinion [Cassandra](http://cassandra.apache.org/)
is a pretty good one.

That being said, if you are building a common web application (i.e Yelp, Feedly, etc.),
you probably have a lot to gain from using RethinkDB.

**Should you use thinky?**  
If you use RethinkDB and Node.js, yes. Thinky works in harmony with RethinkDB
to make writing code easier and faster by doing common things like validation. 

It is easy to learn if you know ReQL since the syntaxes are almost [the same](https://github.com/neumino/thinky/blob/3b4aa9d0fc120c5d99b438328204a3acfa799d1a/lib/query.js#L375).
Despite being simple, thinky is a powerful library:

- Custom validations let you leverage all the power
of libraries like [validator](https://github.com/chriso/validator.js).
- [Asynchronous hooks](http://www.justonepixel.com/thinky/documentation/api/model/#pre) let you use powerful
API like [Mailgun's one](http://documentation.mailgun.com/api-email-validation.html#email-validation)
for email verification.

Give it a shot, and if you have feedback/suggestions, open an issue on [GitHub](https://github.com/neumino/thinky/issues/new),
ping me on Twitter via [@neumino](https://twitter.com/neumino),
or shoot me an email at [orphee@gmail.com](mailto:orphee@gmai.com).
