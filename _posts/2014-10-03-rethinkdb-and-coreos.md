---
layout: post
category : Geek
tags : [rethinkdb, docker, coreos, digitalocean]
title: "RethinkDB and CoreOS: Navigating Digital Ocean Together"
---
{% include JB/setup %}

_A new generation of databases are now sailing on new seas such as Digital Ocean,
transported in Docker containers, and steered by CoreOS._


[CoreOS](https://coreos.com) is a terrific tool to deploy applications on multiple
servers. However, running a [RethinkDB](http://rethinkdb.com) cluster on CoreOS is a bit more
complicated than running multiple Nginx servers with a load balancer since a RethinkDB
instance must be given at least one server to join. This article illustrates one way,
hopefully the right way, to do it on [Digital Ocean](https://digitalocean.com).


First, set up a fleet (or sub-fleet) of CoreOS machines; in this example, we will
suppose that we are building a 6-server RethinkDB cluster. If you have never done it
before, begin by getting a discovery `token` on
[https://discovery.etcd.io/new](https://discovery.etcd.io/new).

Then boot a few CoreOS instances with the following `cloud-config` file and your
ssh keys. Make sure that you enable private networking on Digital Ocean as RethinkDB
does not provide encryption/security for cluster traffic yet.

```
#cloud-config

coreos:
  etcd:
    discovery: https://discovery.etcd.io/<token>
    addr: $private_ipv4:4001
    peer-addr: $private_ipv4:7001
  fleet:
    public-ip: $private_ipv4   # used for fleetctl ssh command
    metadata: group=rethinkdb
  units:
    - name: etcd.service
      command: start
    - name: fleet.service
      command: start
```

Note that these machines will be tagged with the metadata `group=rethinkdb`. I
personally appreciate being able to group my servers depending on
what their responsibilities are (or will be).

To create a RethinkDB cluster, we need to start the instances with the argument
`--join host:port` where another instance of RethinkDB will be running.
We will first create a discovery service where CoreOS servers will register their
IP in [etcd](https://coreos.com/using-coreos/etcd/) using
[etcdctl](https://coreos.com/docs/etcdctl/); we will force RethinkDB to run
on these servers and provide them with all the IP addresses of the servers running the
discovery service.


First, let's create a file `rethinkdb-discovery@.service` on one of your servers with the
following content:

```
[Unit]
Description=Announce RethinkDB@%i service

[Service]
EnvironmentFile=/etc/environment
ExecStart=/bin/sh -c "while true; do etcdctl set /announce/services/rethinkdb%i ${COREOS_PRIVATE_IPV4} --ttl 60; sleep 45; done"
ExecStop=/usr/bin/etcdctl rm /announce/services/rethinkdb%i

[X-Fleet]
X-Conflicts=rethinkdb-discovery@*.service
MachineMetadata=group=rethinkdb
```

Then load the service with:

```
fleetctl submit rethinkdb-discovery@.service
```

Finally, start it with:

```
fleetctl start rethinkdb-discovery@{1..6}.service
```

Now create the service for RethinkDB:

```
[Unit]
Description=RethinkDB@%i service
After=docker.service
BindsTo=rethinkdb-discovery@%i.service

[Service]
EnvironmentFile=/etc/environment
TimeoutStartSec=0
ExecStartPre=-/usr/bin/docker kill rethinkdb%i
ExecStartPre=-/usr/bin/docker rm rethinkdb%i
ExecStartPre=-/usr/bin/mkdir -p /home/core/docker-volumes/rethinkdb
ExecStartPre=/usr/bin/docker pull dockerfile/rethinkdb
ExecStart=/bin/sh -c '/usr/bin/docker run --name rethinkdb%i   \
    -p ${COREOS_PRIVATE_IPV4}:8080:8080                        \
    -p ${COREOS_PRIVATE_IPV4}:28015:28015                      \
    -p ${COREOS_PRIVATE_IPV4}:29015:29015                      \
    -v /home/core/docker-volumes/rethinkdb/:/data/             \
    dockerfile/rethinkdb rethinkdb --bind all                  \
    --canonical-address ${COREOS_PRIVATE_IPV4}                 \
    $(/usr/bin/etcdctl ls /announce/services |                 \
        xargs -I {} /usr/bin/etcdctl get {} |                  \
        sed s/^/"--join "/ | sed s/$/":29015"/ |               \
        tr "\n" " ")'

ExecStop=/usr/bin/docker stop rethinkdb%i

[X-Fleet]
MachineMetadata=group=rethinkdb
X-ConditionMachineOf=rethinkdb-discovery@%i.service
```

The service is first going to fetch data from `etcd` and then start a
[Docker](https://docker.com) container with RethinkDB with the `--join` argument.

Note: Because you are running RethinkDB inside a container, you must provide
the argument `canonical-address` or other instances will try to connect to the
[wrong IP address](https://github.com/rethinkdb/rethinkdb/issues/486).


Run:

```
fleetctl start rethinkdb@{1..6}.service
```

And it's done! You now have a cluster of six machines running RethinkDB.

When RethinkDB provides auto-failover, in the event of a server failure, if you happen to have
an extra CoreOS server, CoreOS will restart another RethinkDB instance and RethinkDB will
automatically re-elect a master/backfill to prepare another
replica without requiring any work on your end.
Heavy refactoring of the clustering is being done right now, so hopefully
this feature should ship soon (~2 months?).

Two things to finish this article:

1. Thanks to [@atnnn](https://github.com/atnnn) for helping me with some bash issues,
[Jessie](https://twitter.com/jessskuo) for proofreading my Frenglish.
2. Questions? Suggestions? Ping me on Twitter: [@neumino](https://twitter.com/neumino).
