---
layout: post
category : Life
tags : [archlinux, lenovo, x1-carbon]
title: Installing Archlinux on a Lenovo X1 Carbon
---
{% include JB/setup %}


I just finished installing/setting up everything on my X1 Carbon, and I have to say, it's just beautiful.

### Installing Archlinux ###

Installing Archlinux on the X1 carbon is pretty straight forward. The X1 Carbon is quite linux-friendly (well, first there is no NVIDIA card).

- Partition your disk with parted (cfdisk doesn't work on the X1 carbon).
    I'm using grub, so I had to use the command "set X boot_grub on" where X is the /boot partition
- Format partition with mkfs.ext2 and mkfs.ext4
- Mount /
- Check your connection. Using the dongle that Lenovo provides works out of the box.
- Set up pacman with 

    ```
    mkdir -pm /mnt/var/lib/pacman
    pacman -r /mnt -Sy base</pre></li>
    ```
- Sign the keys

    ```
    rsync -rav /etc/pacman.d/gnupg/ /mnt/etc/pacman.d/gnupg/
    ```
- Chroot your environment

    ```
    mount --bind /dev /mnt/dev
    mount --bind /sys /mnt/sys
    mount --bind /proc /mnt/proc
    chroot /mnt /bin/bash
    ```
- Edit/fill /etc/fstab
- Setup your hostname, locale, timezone
- Build your initial ram disk

    ```
    mkinitcpio -p linux
    ```
- Install grub
- grub-install --boot-directory=/mnt/boot /dev/sda
grub-mkconfig -o /mnt/boot/grub/grub.cfg</pre>
Add an entry in your grub.cfg file.

```
menuentry "Archlinux" {
    set root=(hd0,X)
    linux /boot/vmlinuz-linux root=/dev/sdaX
    initrd /boot/initramfs-linux.img
}
```
- Reboot and here you go.


### Configuring Archlinux ###

Now your system should boot. I'm providing some tips to configure your X1.
I'm using Gnome (because it's awesome), so if you are not, some tips may not
apply for you.

- The ethernet interface has a strange name (enp0s26u1u2 for me). Run this
command to set up your dongle the first time so you can install a network
manager..

    ```
    ls /sys/class/net
    ip link set XXX up
    ```
- It may not be related to the X1 Carbon, but I had some issues for switching
from a US keyboard to a US International with dead keys keyboard. You can see
<a href="https://bbs.archlinux.org/viewtopic.php?id=157879" title="https://bbs.archlinux.org/viewtopic.php?id=157879">my monologue</a>
on Arch's forums. I end up adding a personal shortcut calling this script

    ```
    #!/bin/bash
    (setxkbmap -query | grep "variant:\s\+intl") && (setxkbmap -model thinkpad -layout 'us') || (setxkbmap -model thinkpad -layout 'us' -variant 'intl')
    ```
- The brightness works on the Gnome settings panel, but my keys did not. After some investigation, Gnome uses the wrong values to set the brighness. I was going to stip out the binary from <a href="http://gitorious.org/gnome-shell-brightness-extension/gnome-shell-brightness-extension" title="http://gitorious.org/gnome-shell-brightness-extension/gnome-shell-brightness-extension">this Gnome extension</a> and bind my FN-keys to it when I realize that the extension could do it for me.
- If you use cpufreq, you'll see your cpu scales from 800Mghz to 1.8 Ghz without going higher (on the governor ondemand). If you use i7z you'll see that the Turbo boost works out of the box and that it's just cpufreq that reports the wrong number.
- To disable the touchpad while typing, I'm using this command (just put it in .profile).
    ```
    syndaemon -d -k -i 0.6s
    ```
- I tried to set up the fingerprint reader (hardware: Upek 147e2020) and with
some pain, I had it working for sudo but not for gdm. I used fingerprint-gui,
and libssapi.so from
<a href="http://volker.de/2012/12/fingerprint-gui-und-das-thinkpad-t430s/bsapi_4-3-291lite_sdk_for_linux-tar/" title="http://volker.de/2012/12/fingerprint-gui-und-das-thinkpad-t430s/bsapi_4-3-291lite_sdk_for_linux-tar/">this archive</a>
and updated  /usr/lib/udev/rules.d/40-libbsapi.rules with the good id. See
<a href="http://volker.de/tag/upek-147e2020/" title="http://volker.de/tag/upek-147e2020/">this page</a>
for more details. Fingerprint-gui broke my Gnome (v 3.6) and I didn't push further.



### Random thoughts

The reason why I bought a X1 Carbon when I have a macbook retina at home is because I found the user experience awful. Everything is not bad on OS X, but there were too many little things that I find annoying, here are some of them:

- Waking up from sleep can take up to 15 seconds.
- When switching virtual desktop, I have to wait for the animation to finish before being able to do something (like opening spotlight to launch an application).
- There is still no native way to manipulate your windows like you can on Windows, Unity or Gnome. And the third parties app I tried were not as good as Gnome.
- I cannot move the first desktop.
- Installing/uninstalling things is pure black magic for me. Maybe that's why people are using <a href="http://mxcl.github.com/homebrew/" title="http://mxcl.github.com/homebrew/">homebrew</a> now.

I used to have an iBook years ago but since I broke it, I switched to a netbook+desktop.

I have good memories of OS X from that time, but since I tried Gnome 3, OS X now just look completely broken.


A macbook is still a nice piece of hardware, I liked the fact that Apple pushed a high definition screen for a laptop, but OS X was too much of a pain for. The aluminium case is nice, but it has two problems

- It's really hot. I cannot let my fingers on the "t" and "y" keys when I'm doing heavy tasks.
- It's really cold when I start using it.

Well anyway, I'll just give my macbook retina to my brother and use my X1 Carbon. I had a great time setting up everything, and it should be the same for using it :)
