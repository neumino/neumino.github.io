---
layout: post
category : Geek
tags : [archlinux, wacom, intuos, bluetooth, linux]
title: Archlinux - Wacom intuos 5 with bluetooth
---
{% include JB/setup %}

I just received yesterday my tablet, a Wacom Intuos 5 medium. Before buying it,
I went through the web and couldn't really figure out if the bluetooth was
working or not. I eventually did bet that it was working and ordered a wireless
kit with my tablet, and the good news is: __It works!__

Installing the driver is quite easy.

```
pacman -S xf86-input-wacom
```

At that time your tablet works with the cable or bluetooth, but the buttons are
not mapped if connected with bluetooth.

Mapping your buttons is a little more tricky.
Gnome 3.6 doesn't work for me, so I just ended up using the xsetwacom command, by
first finding the button's id with xev like explained on
<a href="https://wiki.archlinux.org/index.php/Wacom_Tablet" title="https://wiki.archlinux.org/index.php/Wacom_Tablet">Arch's wiki</a>.
The tricky thing is when you use the bluetooth. For some obscure reasons xev
doesn't catch the event, but that's just xev failing. Find the ids of your buttons
using the cable, then just use xsetwacom and that works. For me the buttons on my
pad are (from top to bottom): 2, 3, 8, 9, 1, 10, 11, 12 and 13.

The command looks something like that:

```
xsetwacom --set "Wacom Wireless Receiver Pen pad" Button 1 "key ctrl z"
```

This settings are not persistent. I quickly tried to use a xorg conf file,
but I just ended up filling my .profile with the commands.

Everything is set now and you can just draw :)
