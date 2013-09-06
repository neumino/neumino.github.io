---
layout: post
category : Geek
tags : [rethinkdb, archlinux, compile]
title: Building RethinkDB from source on Archlinux 
---
{% include JB/setup %}

I just received a Lenovo X1 Carbon and install Archlinux on it.

Since I had a fresh install, I tried to build RethinkDB from source to see what were the dependencies. If you just want to use RethinkDB, you can use <a href="https://aur.archlinux.org/packages/rethinkdb/" title="RethinkDB on AUR">the AUR package</a>.

You will need to install some packages.

```
sudo pacman -S make gcc boost-libs protobuf boost python2 v8 libunwind gperftools java-runtime nodejs
yaourt -S ruby_protobuf
sudo npm install -g coffee-script
sudo npm install -g less
sudo npm install -g handlebars
```

Some libraries are not properly linked when using Archlinux. You need to add these two symbolic links or grep the files that use libprotobuf.a and libprotoc.a and do the appropriates changes.

```
sudo ln -s /usr/lib/libprotobuf.so /usr/lib/libprotobuf.a
sudo ln -s /usr/lib/libprotoc.so /usr/lib/libprotoc.a
```

RethinkDB use python 2 to run some scripts but uses python to refer to python 2. Here is a quick and bad solution.

```
sudo ln -s /usr/bin/python2 /usr/bin/python
```

If you are not on a fresh install, python needs to refer to python 3.
For the time being, you can run a find+sed command to replace all references to python by python2. That's what the PKGBUILD is doing by the way.

```
find ./ -type f -exec sed -i '1 s/python$/python2/' {} +
```

And that's all you need to build RethinkDB from source.
