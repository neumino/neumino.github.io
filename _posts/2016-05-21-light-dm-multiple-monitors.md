---
layout: post
category : Geek
tags : [lightdm, monitor, screen]
title: Lightdm on a specific monitor
---
{% include JB/setup %}

I formatted my workstation yesterday to get rid of Windows for a few reasons:

- The prompt for Windows 10 was too annoying and impossible to remove.
Considering that installing Windows 10 would have blown my dual boot away, and
that I would have to set up my dual boot again, I just went for removing Windows.
- I do not use Photoshop anymore, and found that [Krita](https://krita.org/)
works fine for me.

Anyway, tonight I was trying to get [LightDM](https://wiki.archlinux.org/index.php/LightDM)
to show on my main monitor. I couldn't figure out how to force it on a specific
monitor, but found out that LightDM follows the mouse. So my solution was to
add the following line in `/etc/lightdm/lightdm.conf`:

```
display-setup-script=xdotool mousemove --screen DVI-0 1280 720
```

And that did the trick.
