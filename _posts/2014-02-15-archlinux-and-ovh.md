---
layout: post
category : Geek
tags : [archlinux, ovh, kimsufi]
title: Archlinux on a OVH kimsufi server
---
{% include JB/setup %}

The Archlinux distribution installed on OVH kimsufi servers come with a custom kernel.

While it is not super-old, it is still not up to date (at the time of writing, OVH is
using 3.10.9 while Arch comes with 3,12.9).

Installing Arch from scratch seems to be doable, but is probably some work. Another way
to have a recent kernel is just to use OVH template and then swap the kernel. For that
you just need to:

- Install Arch from the web interface
- Update `/etc/pacman.d/mirrorlist` with normal values (look at `mirrorlist.pacorig`)
- Generate a new grub conf with

```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
- Make sure that the entry with the normal kernel in `/boot/grub/grub.cfg` is the first one.
- Reboot

Then you get:

```
michel@ks******:~$ uname -r
3.12.9-2-ARCH
```

