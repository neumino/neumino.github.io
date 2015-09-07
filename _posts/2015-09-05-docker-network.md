---
layout: post
category : Geek
tags : [docker, network]
title: Docker without sudo, network issues
---
{% include JB/setup %}

If you want to run `docker` without using sudo, you can add yourself to the `docker` group.

```
sudo usermod -aG docker <user>
```

In my case (Archlinux), this was not quite enough. Starting a container would work but
without internet access. The main problem was that I was not part of the `network` groop.

```
sudo usermod -aG network <user>
```

Then things went smoothly.
