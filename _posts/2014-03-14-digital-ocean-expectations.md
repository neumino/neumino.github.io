---
layout: post
category : Geek
tags : [archlinux, digital ocean]
title: First experience on Digital Ocean - Updating Archlinux
---
{% include JB/setup %}

I have been using a dedicated server at OVH for a few years now, and the quality of their
service has become worse and the last incidents prompted me to look for a new server.  
Digital Ocean claims that they are user-friendly and since it is quite cheap, I just
gave it a try.

Subscribing, setting up 2 factor authentification, starting a droplet was a blast. I
picked Archlinux, and less than one minute after, my droplet was up and running.


The Arch image is quite old (June 2013) and updating the system is a little more tricky
than just running `pacman -Syu`.  
These instructions were written a few hours after the installation, so they may be
slightly inaccurate.


First, update the whole system. Because Arch
[merged](https://www.archlinux.org/news/binaries-move-to-usrbin-requiring-update-intervention/)
`/bin`, `/sbin` into `/usr/bin` and `/lib`
into `/usr/lib`, you cannot just run `pacman -Syu`. Run instead:

```
pacman -Syu --ignore filesystem,bash
pacman -S bash
pacman -Su
```

Then remove `netcfg` and install `netctl`.

```
pacman -R netcfg
pacman -S netctl
```

Run `ip addr` to see your interface. In my case it was `enp0s3`

Create a config file `/etc/netctl/enp0s3` with

```
Interface=enp0s3
Connection=ethernet
IP=static
Address=('<droplet_ip>/24')
Gateway='<gateway>'
DNS=('8.8.4.4', '8.8.8.8')
```

Enable the interface

```
netctl enable enp0s3
```

Then update the kernel via the web interface.

The network interface is going to change to something like `ens3`. Move
`/etc/netctl/enp0s3` to `/etc/netctl/ens3` and change the `Interface` field.

Update `/lib/systemd/system/sshd.service` to be sure that the ssh daemon doesn't
[fail on boot](https://wiki.archlinux.org/index.php/Secure_Shell#Managing_the_sshd_daemon)

```
[Unit]
Description=OpenSSH Daemon
Wants=sshdgenkeys.service
#After=sshdgenkeys.service
After=network.target

[Service]
ExecStart=/usr/bin/sshd -D
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=always

[Install]
WantedBy=multi-user.target
```

Reboot and your server should be up to date.

And that's it for updating Arch. It was not the easiest updates, but nothing impossible.
It would have been nice if Digital Ocean was provided an up to date Arch image though.

-------
Note: You can probably directly set the network interface to `ens3`.  
In the worst case you can still access your machine with Digital Ocean's web shell and fix things there.
