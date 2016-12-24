---
layout: post
category : Geek
tags : [kubernetes]
title: Docker fails to create endpoint
---
{% include JB/setup %}

I ran into this issue recently while trying to get tensorflow running:

```
michel@xone:~$ docker run  -p 8888:8888 gcr.io/tensorflow/tensorflow
docker: Error response from daemon: failed to create endpoint high_raman on network bridge: failed to add the host (veth3c0e4fe) <=> sandbox (veth21faddb) pair interfaces: operation not supported.
```

Nothing looked suspicious in `ip addr`

```
michel@xone:~$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: wlp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 8c:70:5a:ff:a4:08 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.71/24 brd 192.168.1.255 scope global dynamic wlp3s0
       valid_lft 79660sec preferred_lft 79660sec
    inet6 2602:306:ce40:a090:8e70:5aff:feff:a408/64 scope global noprefixroute dynamic 
       valid_lft 2591806sec preferred_lft 604606sec
    inet6 fe80::8e70:5aff:feff:a408/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:d3:0f:49:9b brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
```

Looking around, it
[seems](https://github.com/docker/docker/issues/15341#issuecomment-218930712)
that Archlinux needs to reboot for Docker to properly work. I didn't look any
further, but at least this did the trick for me.
