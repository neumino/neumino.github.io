---
layout: post
category : Geek
tags : [archlinux, ttf-dejavu]
title: Force update ttf-dejavu
---
{% include JB/setup %}

On archlinux, ttf-dejavu 2.37 requires a forced upgrade. If you don't, you will be greeted
with this error:

```
michel@xone:~$ sudo pacman -S ttf-dejavu
[sudo] password for michel: 
resolving dependencies...
looking for conflicting packages...

Packages (1) ttf-dejavu-2.37-1

Total Installed Size:  9.75 MiB
Net Upgrade Size:      0.57 MiB

:: Proceed with installation? [Y/n] Y
(1/1) checking keys in keyring                                                                             [###############################################################] 100%
(1/1) checking package integrity                                                                           [###############################################################] 100%
(1/1) loading package files                                                                                [###############################################################] 100%
(1/1) checking for file conflicts                                                                          [###############################################################] 100%
error: failed to commit transaction (conflicting files)
ttf-dejavu: /etc/fonts/conf.d/20-unhint-small-dejavu-sans-mono.conf exists in filesystem
ttf-dejavu: /etc/fonts/conf.d/20-unhint-small-dejavu-sans.conf exists in filesystem
ttf-dejavu: /etc/fonts/conf.d/20-unhint-small-dejavu-serif.conf exists in filesystem
ttf-dejavu: /etc/fonts/conf.d/57-dejavu-sans-mono.conf exists in filesystem
ttf-dejavu: /etc/fonts/conf.d/57-dejavu-sans.conf exists in filesystem
ttf-dejavu: /etc/fonts/conf.d/57-dejavu-serif.conf exists in filesystem
Errors occurred, no packages were upgraded.
```

Make sure to run pacman `-f` for this package.
```
michel@xone:~$ sudo pacman -S --force ttf-dejavu
```
