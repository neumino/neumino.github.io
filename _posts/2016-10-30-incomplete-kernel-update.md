---
layout: post
category : Geek
tags : [archlinux, kernel, encryption, disk]
title: Incomplete kernel update
---
{% include JB/setup %}

During a system update, my workstation suddenly shut down (hardware failure). I
was working on something else during the update, so I had no idea when the
update was aborted. When I tried to boot again, I was greeted with this error:

```
Warning: /lib/modules/4.8.4-1-ARCH/modules.devname not found - ignoring.
starting version 231
ERROR: device 'UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx' not found, Skipping fsck.
ERROR: Unable to find root device 'UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'.
You are being dropped to a recovery shell
    Type 'exit' to try and continue booting
sh: can't access tty: job control turned off
[rootfs ]#
```

At that time I thought that one of my harddrive died. Digging a bit into it,
the partition that couldn't be found was an encrypted one. So I thought that
maybe my fstab got somehow corrupted.

I didn't have access to a shell after booting as the keyboard was not working,
so I just booted with a live usb key. My disks are all encrypted, so the first step was to decrypt them:


Confirm that they are indeed crypted:

```
blkid | grep crypto
```

Decrypt/mount the partition:

```
cryptsetup luksOpen /dev/sdc2/ crypt
```

You then can't just mount `/dev/mapper/crypt`. If you try, you'll get:

```
mount /dev/mapper/crypthome /mnt/crypt
mount: unknown filesystem type 'LVM2_member'
```

Find the group of your volume (mine was main):

```
vgscan
```

Find the name of the volume (mine was root):

```
lvs
```

Mount it:

```
mount /dev/main/root /mnt
```

In my case, `/boot` is not encrypted, so I also had to mount it to be able to
look at my kernel.

```
mount /dev/sdc1 /mnt/boot
```

Then I just used chrooted:

```
arch-chroot /mnt
```

I thought that the issue might have been in the initial ramdisk environment, so I just ran:

```
mkinitcpio -p linux
```

I was greeted with quite a few warnings that some modules were not found. But
all the modules for encryption was present, so I decided to just reboot.
Some progress were made: I could decrypt my disk, but was then greeted
with:

```
mount: unknown filesystem type 'ext4'
```

I booted on the usb key again, and looked a bit at my modules. That's when I found
that `uname -r` was giving me an old kernel version. Trying to reinstall the `linux`
package was returning an error because the `desc` file was missing. I guess my
update was interrupted during the kernel update.

Removing the package and re-installing it did the trick. Note that `pacman -S --force`
was not enougth to prevent pacman from complaining about the missing files.

```
pacman -R linux
pacman -Sy linux
mkinitcpio -p linux
```
