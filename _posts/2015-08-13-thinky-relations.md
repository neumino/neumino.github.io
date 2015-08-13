---
layout: post
category : Geek
tags : [thinky, relation, rethinkdb]
title: "Thinky 2.1.1 - Updating only relations"
---
{% include JB/setup %}

I released thinky 2.1.1 a bit earlier today.

Thinky was built on the assumption that you always retrieve all the documents from
the database because updating a relation. Typically this was the expected workflow.

```js
var thinky = require('thinky')();
var type = thinky.type();

User = thinky.createModel("User", {
  id: type.string(),
  email: type.string().required(),
  name: type.string().required(),
  adult: type.boolean().default(false)
}

User.hasAndBelongsToMany(User, "friends", "id", "id");

User.get("3851d8b4-5358-43f2-ba23-f4d481358901")
    .getJoin({friends: true}).run().then(function(user) {
  // user is fully defined with its friends
  user.friends = [];
  return user.saveAll({friends: true});
}).then(function(user) {
  // user 3851d8b4-5358-43f2-ba23-f4d481358901 has no more friends!
});
```

Looking at the GitHub issue tracker, many developers had issues. Trying to
save a relation with pre-existing documents can throw errors like [#245](https://github.com/neumino/thinky/issues/245)
and basically wasn't clear for many [#309](https://github.com/neumino/thinky/issues/309).
Thinky 2.1.1 takes a stab at all these problems and introduces two new commands:

```
 - addRelation(field, joinedDocument)
 - removeRelation(field[, joinedDocument])
```

You can chain these commands on a query that returns one document and add/remove a relation.


_Example:_ Add a new friend

```js
User.hasAndBelongsToMany(User, "friends", "id", "id");

User.get("3851d8b4-5358-43f2-ba23-f4d481358901")
    .addRelation('friends', {id: '0e4a6f6f-cc0c-4aa5-951a-fcfc480dd05a'})
    .run()
```

_Example:_ Remove a new friend

```js
User.get("3851d8b4-5358-43f2-ba23-f4d481358901")
    .removeRelation('friends', {id: '0e4a6f6f-cc0c-4aa5-951a-fcfc480dd05a'})
    .run()
```

_Example:_ Remove all friends

```js
User.get("3851d8b4-5358-43f2-ba23-f4d481358901")
    .addRelation('friends')
    .run()
```

The second argument to `addRelation` needs to provide just enough information to create a relation.

- In the case of a `hasOne` or `hasMany` relation, only the primary key is required.
- In the case of a `belongsTo` or `hasAndBelongsToMany` relation, the primary key or the right key is required - one is enough!

The same argument for `removeRelation` is not always required and if it is defined, it also needs just enough information
to find the relation to delete.

- For `hasOne` and `belongsTo` relations, the `joinedDocument` is not required and should not be provided.
- For `hasMany` and `hasAndBelongsToMany` relations, the absense of the `joinedDocument` makes the command
delete all the relations for the given document. If a document is provided, the primary key or the right key
is required - again, one of them is enough.

This update should enable people easier to manipulate relations without the need to retrieve the documents
from the database. Beyond a simpler interface, it should also ease a little the load on the database.

Feedback/suggestions? Ping me on Twitter via [@neumino](https://twitter.com/neumino)
or shoot me an email at [orphee@gmail.com](mailto:orphee@gmai.com).
