---
layout: post
category : Geek
tags : [thinky, rethinkdb, date]
title: Thinky v0.2.15 - But what is it?
---
{% include JB/setup %}

I just updated [thinky](https://github.com/neumino/thinky) with dates and just
thought that it would be nice to jump on the occasion to talk a little about this
project.


### What is it? ###

Thinky is a JavaScript ORM for RethinkDB.

### Other JavaScript ORM ###

As far as I know there is just one other JavaScript ORM for RethinkDB, which is
a plugin for [jugglingdb](https://github.com/1602/jugglingdb) named [jugglingdb-rethink](https://github.com/fuwaneko/jugglingdb-rethink).
The goal of jugglingdb is to provide an ORM with one syntax that would
work on many databases, including MySQL, Redis, MongoDB etc.

I tend to think that ReQL has a really great API - but that may be my biased point
of view. Anyway, I would rather not use an API that uses JSON
to represent relations like `in` or logic operators like `and`. Thinky aims to
provide an API as nice as the official Node driver.

### So what is in the box? ###

Thinky aims to stick close to the official node driver. A few differences are:

- You can set default values when creating an object (including functions
that would be called when the object is being created), so if you want to
store the date at which an object is created, you can use this schema

    ```js
    var Cat = thinky.createModel('Cat',
        {
            name: String,
            createdAt: {_type: Date, default: function() { return new Date() }}
        }); 
    var kitty = new Cat({
        name: "Kitty"
    });
    // kitty will have a field `createdAt`.
    ```

- You can pass the callback directly to the last method instead of using the
`run` command. So these two syntax are the same

    ```js
    Cat.get('3851d8b4-5358-43f2-ba23-f4d481358901', callback);
    Cat.get('3851d8b4-5358-43f2-ba23-f4d481358901').run(callback);
    ```


- You can define methods on your schemas, making your code look more like a
object oriented program.

    ```js
    var Cat = thinky.createModel('Cat', { name: String }); 
    Cat.define('sayHello', function() { console.log("Hello, I'm "+this.name) });
    var kitty = new Cat({ name: "Kitty" });
    kitty.sayHello()

    ```

- Because you define schemas and relations, doing a `JOIN` operation is as
simple as calling the `getJoin` method.

    ```js
    Cat = thinky.createModel('Cat', {id: String, name: String});
    Task = thinky.createModel('Task', {id: String, task: String, catId: String});
    Cat.hasMany(Task, 'tasks', {leftKey: 'id', rightKey: 'catId'});

    Cat.get( 'b7588193-7fb7-42da-8ee3-897392df3738').getJoin( function(err, result) {
        // Returns a cat with its tasks
    })
    ```

_Note_: Some limitations exists about `getJoin` now and should be solved once
the proposal described [here](https://github.com/neumino/thinky/issues/19#issuecomment-24032730)
will be implemented.

### Awesome, how do I get it? ###

There is an npm package, just run

```
npm install thinky
```

And you should be all set.

### Nice, how can I help? ###

Any bug reports, suggestions, pull requests (code or docs) are more than welcome!
