---
layout: post
category : Geek
tags : [pandora, leapcast, chromecast, sound, pulseaudio]
title: Quick hack - Sound volume on Leapcast 
---
{% include JB/setup %}

Leapcast currently does let you change the sound volume with Pandora (see the
[source](https://github.com/dz0ny/leapcast/blob/master/leapcast/services/websocket.py#L317)
for `CastPlatform`).

I did a quick/dirty hack to change the sound volume on my computer with the `pactl` command.
That works only if you are running pulseaudio (tested only with Linux for now).

This [commit](https://github.com/neumino/leapcast/commit/6bb7b227ef3b772f997c76bf9d9e50773db5eb43)
let you change the sound volume (if you are running pulseaudio).  

You may have to set the `--pulse` argument to make it work. Run `pacmd list` to find your
running pulseaudio server.  
You can provide the index (an integer) or the name of the server (like
`alsa_output.pci-0000_00_1b.0.analog-stereo`).
