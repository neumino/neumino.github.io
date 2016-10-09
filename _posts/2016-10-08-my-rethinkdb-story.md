---
layout: post
category : Life
tags : [RethinkDB]
title: My RethinkDB story
---
{% include JB/setup %}

RethinkDB recently [announced](https://rethinkdb.com/blog/rethinkdb-shutdown/)
that the company behind the database is shutting down. In the last few days,
many people have started to write about RethinkDB; the
[article about open source companies](http://sagemath.blogspot.com/2016/10/rethinkdb-sagemath-andreessen-horowitz.html)
from William Stein, author of SageCloud, is probably the most interesting.
Today, I would like to share a different story - my personal one with RethinkDB. 

After graduating from Tsinghua (Beijing), I was looking for a new country to live in and
eventually settled on moving to California. I sent a plethora of emails and eventually
participated in a [HackerRank](https://hackerrank.com) competition where I
ranked 17th (or was it 13th?). I then got the opportunity to interview at RethinkDB.
After a few calls, I got a job offer from RethinkDB; this was my first job.

My first days at RethinkDB were wonderful. We were developing on a local/shared
server because building RethinkDB is quite CPU intensive. This is how I really
learned how to quit vim. One of my most vivid memory of that time is the
day Slava asked me if we should just fork CoffeeScript (CS) after getting
bitten by another bug - This was in CS early-ish days, version 1.3.x. While we
eventually did not end up forking CS, Slava's suggestions felt like an easy
thing to do and made me realize that while I was a good programmer, I had much
more to learn.

Thosee early days were also great because RethinkDB, as a NosQL database, was not
yet public. We were moving extremely fast without doing code review, while
maintaining a high code quality, thanks to a strong sense of ownership.
The web interface became more polished and the data explorer turned out amazing
once we added suggestions and auto completion.

Launch day was incredible. We spent the whole day just answering questions.
The feedback on the web interface was extremely positive. To this day, I still
sometime wonder if the web interface was too shiny; people did not realize
how great the database engine under the hood was. The tremendous amount of
positive feedback just fueled our passion even more. We kept working hard on
it, polishing edges, adding new features etc.

We had great discussions about ReQL. One of the most interesting one I can
remember was about the `group` command that replaced `groupedMapReduce`. We
all had different opinions on what would be best for our users - I wanted the
scope of chained command to be always the same as the parent. The sheer amount
of thoughts poured into ReQL made it the best query language available. Nowadays,
a little part of me cries every time I have to write a SQL query for work.

Jessie started to work at RethinkDB about 2 years after I joined. We had a lot
of fun. I couldn't help but be amused when I learned that she ate a box full of
bacon for breakfast. We started to date after a few months and got engaged a
few weeks ago!

After about 2 years and a half, I made the hard decision to leave the team
for another company. Even though I was not employed by RethinkDB (technically
Hexagram 49 Inc), I always kept RethinkDB in my heart. I kept maintaining
rethinkdbdash and thinky, refactored reqlite to support `r.http` which involved
making all the internal operation asynchronous etc.

Horizon then came out, and boy, that was a really good project. I couldn't help
but notice that it wasn't gaining traction fast enough, and the sad news that
RethinkDB was shutting down eventually came out. While the company is gone, I
am convinced that the database will live beyond. This is nothing more than a new
chapter for RethinkDB and I am excited to see what will happen next.

Slava, Michael, all the team, I took pride in crafting part of RethinkDB, in
supporting our users, and I enjoyed every bit of sailing with all of you.
My time at RethinkDB was amazing; I have nothing but great memories from
working with all of you.
