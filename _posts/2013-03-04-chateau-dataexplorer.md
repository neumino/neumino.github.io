---
layout: post
category : Geek
tags : [chateau, rethinkdb, dataexplorer]
title: Chateau - A data explorer for RethinkDB
---
{% include JB/setup %}

### A little story

Two weeks ago I started to bootstrap a new idea. At some point I had to write an administration interface to manage my data. That was just a huge pain. So I decided to write one that I could easily reuse and that is how <a href="https://github.com/neumino/chateau" title="https://github.com/neumino/chateau">Chateau</a> was born.

One of the reason why people love frameworks like Django is because you just have to create the schema of your table, and Django provides you with a nice http interface to add/edit/delete rows. So I just thought about doing something similar with RethinkDB with two small differences.

- The web has tons of different stacks so I wrote Chateau to be independent of your stack. It is a standalone node.js server. Another reason to have an standalone server is because RethinkDB doesn't provide authentification (see <a href="https://github.com/rethinkdb/rethinkdb/issues/266" title="Consider a design for access control">issue</a>) so people may want to take down their admin interface when they are done bootstrapping.
- The last notable difference is that when bootstrapping a project, your data's schema might change and you probably do not want to change a config file every time, so I made Chateau to infer your schema itself.


### What's in the box?
Well sorry, there is not really a box, you have to build from source. You need to install node and the following libraries: <em>express, coffee-script, handlebars, stylus, rethinkdb</em>. I will do some packaging later if people are interested in.

### So what's in the source?
The alpha version comes with these features:
- It list all your databases and tables and let you do some operations on it (creating and deleting).
Renaming a database or a table can not be done with the driver (RethnkDB 1.3.2), so I will wait for 1.4 to be released instead of hacking something with the http api.
- It shows 1000 documents in a table (pagination is not yet available). The first column is the primary key and the following one are ordered (the most shared attribute is first).
- You can add a document. Chateau will infer the schema and ask you just to fill some inputs. If you disagree with the proposed schema, changing it is just about toggling a select tag.
- You can update/delete a document.

### Cool, what's next?
Can be done anytime depending on how much time I have:

- Fix some css edges cases
- Do the main TODOs in the code
- Refactor the code for add/update a document
- Implement/test all errors handling
- Improve Makefile
- Add loading for logout
- Change the icon
- Screenshots/screencast

Waiting for RethinkDB 1.4 (hopefully this week)

- Update Chateau to work with the new API of the JavaScript driver
- Sort the documents
- Filter documents

Advanced features that I am interested in implementing because they are tricky and no one did it before.

- Support joins
- Add custom actions
- Add custom views 

Things that may never be done, but who knows?

- Handle multiple RethinkDB instances at the same time
- Support https


If you are interested to join/want to provide some feedback, you are welcome!
Ping me at orphee@gmail.com, neumino on irc.freenode.org, @neumino on Twitter.
