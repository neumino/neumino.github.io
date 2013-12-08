---
layout: post
category : Geek
tags : [rethinkdb, raspberry, pi, arm]
title: Building RethinkDB on a Raspberry Pi
---
{% include JB/setup %}

It took 3 whole days, but I did build [RethinkDB](http://rethinkdb.com) on my [
Raspberry Pi](http://www.raspberrypi.org/), and it's working!

All the praise should go to [@davidthomas426](https://github.com/davidthomas426) who
submitted a [pull request](https://github.com/rethinkdb/rethinkdb/pull/1625) for
RethinkDB to support ARM.


Here is what I have done to build RethinkDB on a Raspberry Pi.  
A few things about these instructions:

- There may be a few quirks as I wrote them after building RethinkDB.
- They are slightly different than what you can find on
[RethinkDB's website](http://www.rethinkdb.com/docs/install/arch/) since the branch I
used was based on 1.10.

I built RethinkDB on [Archlinux Arm](http://archlinuxarm.org/platforms/armv6/raspberry-pi).  
Building RethinkDB requires more than 500MB of RAM, so you have to create a swap
partition. I used a 2GB swap, a smaller swap may work (I would say 1GB is enough), but 
I haven't tried it.

Install some dependencies.

```
sudo pacman -S make gcc protobuf boost python2 gperftools nodejs base-devel python2-pip
```

Make `python2` the default `python`.

```
sudo rm /bin/python
sudo ln -s /bin/python2 /usr/python
```

Install `pyyaml`.

```
sudo pip2 install pyyaml
```

Install `v8` from AUR.

```
yaourt -S v8
```

Clone the source

```
git clone https://github.com/davidthomas426/rethinkdb
cd rethinkdb
git checkout davidthomas426_277_arm_support
```

Run `configure`

```
./configure --dynamic tcmalloc_minimal
```

I also changed the swappiness to `10` - I'm not sure how useful it is though.  

```
sudo sysctl vm.swappiness=10
```

You have to build with `DEBUG=1` to avoid
[this bug](https://github.com/rethinkdb/rethinkdb/issues/1731) (it will be fixed in
1.12)

```
make DEBUG=1
```
You may see warnings like `note: the mangling of 'va_list' has changed in GCC 4.4`. You
can just ignore those.

After about 3 days, you can start RethinkDB with

```
./rethinkdb -c 1 -no-direct-io
```

If you are looking for the binary, it's available [here](http://justonepixel.com/retihnkdb/pi/20131206rethinkdb)

I will try to spend some time creating a branch based on `next` that supports ARM and
build again.
