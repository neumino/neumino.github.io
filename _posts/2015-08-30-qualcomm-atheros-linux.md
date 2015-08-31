---
layout: post
category : Geek
tags : [archlinux, qualcomm, atheros AR8161, bug]
title: Losing connectivity after a Chrome/Firefox request. 
---
{% include JB/setup %}

When I upgraded my kernel from 4.0.7-2 to 4.1.4.1 (and more recently to 4.1.6-1)
I had a really annoying bug and since it wasted some of my time, I hope that this
post may help someone.

After my 4.1 kernel update, I would start connected to the internet. I could ping any server,
use `wget`, but as soon as I would make a request on Chromium or Firefox, I would "lose
connectivity" after a few seconds. By loosing connectivity, the pings would timeout or fail with "Destination
Host Unreachable". Once of the reason I had a hard time finding the reason is that I had
no errors reported by the kernel or the driver. The only way to get back a connectivity was to
take down the interface and bring it back up.

Eventually, everything boiled down to my Qualcomm Atheros AR8161 driver. Increasing the MTU
to 9000 fixed my problem as described on this [archlinux bug](https://bugs.archlinux.org/task/44315)
