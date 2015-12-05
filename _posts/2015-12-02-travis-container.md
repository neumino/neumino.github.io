---
layout: post
category : Geek
tags : [travis, container, package]
title: Using Travis'container architecture
---
{% include JB/setup %}

Last year, Travis [announced](https://blog.travis-ci.com/2014-12-17-faster-builds-with-container-based-infrastructure) 
faster builds with container-based infrastructure and Docker.

One requirement to use such infrastructure is to disable `sudo` with `sudo: false`
at the top of your `.travis.yml` file.

If you were installing third party software using `apt-get` and `sudo`, you can
just download the `.deb` package, and unpack it. For RethinkDB, you can run these
commands:

```
wget http://download.rethinkdb.com/apt/pool/precise/main/r/rethinkdb/rethinkdb_2.2.1
ar x *.deb
ar xvzf data.tar.gz
```

You can then find your binary in `./usr/bin`. For example, this is reqlite
[.travis.yml](https://github.com/neumino/reqlite/blob/master/.travis.yml):


```
language: node_js
sudo: false
node_js:
  - "node"
before_install:
  - wget http://download.rethinkdb.com/apt/pool/precise/main/r/rethinkdb/rethinkdb_2.2.1~0precise_amd64.deb
  - ar x *.deb
  - tar xvzf data.tar.gz
before_script:
  - ./usr/bin/rethinkdb --daemon
  - npm install
  - ./bin/reqlite --port-offset 1 &
  - sleep 10
after_script:
  - rethinkdb
notifications:
  email: false
```
