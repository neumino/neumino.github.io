---
layout: post
category : Geek
tags : [rethinkdb, python, protobuf]
title: Python driver for RethinkDB with protobuf 2.5
---
{% include JB/setup %}

The python driver for RethinkDB has two implementations of the Google protobuf protocol.
One is written in pure python and one in C++. The C++ back-end is really faster (See the
[1.7 release post](http://rethinkdb.com/blog/1.7-release/)).

The protobuf files provided in the pip package are compiled with protobuf 2.4 (version
currently supported by Ubuntu). So if you are using a more recent version of protobuf
(like if you are on Arch), you will see this error

```
In file included from ./rethinkdb/ql2.pb.cc:4:0:
./rethinkdb/ql2.pb.h:17:2: error: #error This file was generated by an older version of protoc which is
#error This file was generated by an older version of protoc which is
^
./rethinkdb/ql2.pb.h:18:2: error: #error incompatible with your Protocol Buffer headers. Please
#error incompatible with your Protocol Buffer headers.  Please
^
./rethinkdb/ql2.pb.h:19:2: error: #error regenerate this file with a newer version of protoc.
#error regenerate this file with a newer version of protoc.
^
*** WARNING: Unable to compile the C++ extension
command 'gcc' failed with exit status 1
*** WARNING: Defaulting to the python implementation
```

You can create a package with the appropriate protobuf file if you clone rethinkdb, but
if you are lazy, here is the package with the protobuf 2.5 files.

- [http://justonepixel.com/rethinkdb/rethinkdb-1.10.0-0-protobuf-2.5.tar.gz](http://justonepixel.com/rethinkdb/rethinkdb-1.10.0-0-protobuf-2.5.tar.gz)


Install it with

```
wget http://justonepixel.com/rethinkdb/rethinkdb-1.10.0-0-protobuf-2.5.tar.gz
sudo pip install rethinkdb-1.10.0-0-protobuf-2.5.tar.gz
```

It it worked fine, this command should print "cpp"

```
python -c "import rethinkdb as r; print r.protobuf_implementation"
```
