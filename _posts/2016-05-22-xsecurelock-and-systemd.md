---
layout: post
category : Geek
tags : [xsecurelock, google, systemd]
title: Lock with XSecureLock after suspend
---
{% include JB/setup %}

To lock your computer with XSecureLock after suspend, create a new systemd
service at `/usr/lib/systemd/system/xsecurelock@.service`

```
[Unit]
Description=Lock X session using xsecurelock
After=suspend.target

[Service]
Type=simple
Environment=DISPLAY=:0
Environment=XAUTHORITY=/home/%i/.Xauthority
ExecStart=/usr/bin/xsecurelock auth_pam_x11 saver_blank

[Install]
WantedBy=suspend.target
```

Enable it with `sudo systemctl enable xsecurelock@<user>.service`.

Then make sure you suspend your computer with `systemctl`
(`sudo systemctl suspend`) and not directly with `sudo pm-suspend`. If you
don't systemd won't trigger your service after suspend.
