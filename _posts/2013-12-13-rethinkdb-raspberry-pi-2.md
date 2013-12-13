---
layout: post
category : Geek
tags : [rethinkdb, raspberry, pi, arm]
title: Building RethinkDB on a Raspberry Pi - Part 2
---
{% include JB/setup %}

I previously wrote about [compiling RethinkDB](http://blog.justonepixel.com/geek/2013/12/06/rethinkdb-raspberry-pi/) on a Raspberry Pi.

I merged [@davidthomas426](https://github.com/davidthomas426)'s branch in the branch
`next` of RethinkDB (based on v1.11), and things seem to still work.  
You can get this branch here: [michel_arm](https://github.com/neumino/rethinkdb/tree/michel_arm).  


Instructions to build are still the same, except that you do not need PyYaml anymore.  
So far the only thing that doesn't work (and that I am aware of) is the flag `--bind all`.


I'll be going home next week (and won't travel with my raspberry pi), so I'll try to spend some this week end to fix this `--bind all` flag issue.
