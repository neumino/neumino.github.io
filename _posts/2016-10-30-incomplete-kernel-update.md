---
layout: post
category : Geek
tags : []
title: Incomplete kernel update
---
{% include JB/setup %}


blkid | grep crypto
cryptsetup luksOpen /dev/sdc2/ crypt
mkdir /mnt/crypt

mount /dev/mapper/crypthome /mnt/crypt

mount: unknown filesystem type LVM2_member


group
$ sudo vgscan
name
$ sudo lvs
Create a mount point for that volume:
$ sudo mkdir /mnt/fcroot
Mount it:
$ sudo mount /dev/VolGroup00/LogVol00 /mnt/fcroot -o ro,user
Copied my files.

