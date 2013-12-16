---
layout: post
category : Geek
tags : [pandora, leapcast, chromecast]
title: Pandora on leapcast
---
{% include JB/setup %}

I looked for a way to start/control Pandora on my desktop computer from my phone.

Googling around I found about [Pianobar](https://github.com/PromyLOPh/pianobar) and
[Pianobar Remote](https://play.google.com/store/apps/details?id=com.pianobar.remote&hl=en).

Pianobar Remote sends commands to Pianobar via ssh, so it requires some credentials to ssh
in your machine. I am somehow not a big fan of granting an app access to my
computer, especially when it is not open source, so Pianobar/Pianobar Remote was not a viable setup for me.

I then found about [Leapcast](https://github.com/dz0ny/leapcast), a ChromeCast emulation
app. However Leapcast does not support Pandora.

I hacked my way around and now it works enough for me to start/switch music from my couch
while sipping a cup of coffee and reading a book.

What's working now:

- Start/stop Pandora
- Play, skip, vote

What doesn't work:

- Sound control

I also added a little thing to make Leapcast discoverable for only a set of ips. That's useful if you
have roomates that love pranks :)


See the [pull request](https://github.com/dz0ny/leapcast/pull/69) for more info.  
I also wrote a little file for systemd.

Content of `/usr/lib/systemd/system/leapcast.service`:

```
#  This file is for leapcast.

[Unit]
Description=Start leapcast

[Service]
Type=simple
ExecStart=/usr/bin/leapcast --chrome /usr/bin/chromium --name Gratin --ips 192.168.69.125,192.168.32.112
KillMode=process
User=michel
Group=michel
TTYPath=/dev/tty1
Environment=DISPLAY=:0
Restart=on-failure
RestartSec=60

[Install]
WantedBy=multi-user.target
```

Then run

```
sudo systemctl enable leapcast
```
