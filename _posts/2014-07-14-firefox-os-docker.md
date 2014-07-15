---
layout: post
category : Geek
tags : [rethinkdb, nodejs]
title: Docker container for Firefox OS
---
{% include JB/setup %}

I recently got a [Flame](https://developer.mozilla.org/en-US/Firefox_OS/Developer_phone_guide/Flame), the
developer reference phone for Firefox OS.

I created a Docker container to build Firefox OS, mostly because I didn't feel like installing Java 6 on my system (since it is
[not supported anymore](http://www.oracle.com/technetwork/java/eol-135779.html)). This post is about how to build such the container.


If you are interested in the image, you can find it on the 
[Docker's hub](https://registry.hub.docker.com/u/neumino/ubuntu-firefoxos/) once [this issue](https://github.com/dotcloud/docker/issues/7034) will be solved.

Start it with

```
sudo docker run -t -i --privileged --expose 5037 -v /dev/bus/usb:/dev/bus/usb -v /host/data:/container/data ubuntu-firefoxos /bin/bash
```


__Steps to create the container:__

Start a Ubuntu container.

```
sudo docker run -t -i ubuntu:14.04 /bin/bash
```

Update the system.

```
apt-get update
apt-get upgrade
```

Install the dependencies as described [in the docs](https://developer.mozilla.org/en-US/Firefox_OS/Firefox_OS_build_prerequisites#Ubuntu_13.10).

```
dpkg --add-architecture i386
apt-get update
apt-get install --no-install-recommends autoconf2.13 bison bzip2 ccache curl flex gawk gcc g++ g++-multilib gcc-4.6 g++-4.6 g++-4.6-multilib git lib32ncurses5-dev lib32z1-dev zlib1g:amd64 zlib1g-dev:amd64 zlib1g:i386 zlib1g-dev:i386 libgl1-mesa-dev libx11-dev make zip libxml2-utils
apt-get install python
apt-get install android-tools-adb
update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.6 1 
update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 2 
update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-4.6 1 
update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-4.8 2 
update-alternatives --set gcc "/usr/bin/gcc-4.6" 
update-alternatives --set g++ "/usr/bin/g++-4.6" 
```

__Note__: All the instructions below are what I did to be able to build the OS for the Flame. Some steps may not be required for another
phone (and some may be missing).

You need to specify some credential for `git`.

```
git config --global user.email <your_email>
git config --global user.name <your_name>
```

Then you have to install a few more things to be able to build, with first Java.

```
add-apt-repository ppa:webupd8team/java
apt-get update
install oracle-java6-installer
```

Building Firefox OS requires you to pull some blobs from your phone with `adb`.

```
apt-get install android-tools-adb
apt-get install libusb-1.0-0 libusb-1.0-0-dev
apt-get install usbutils # This may not be needed, I used it to debug a few things
```

Install a few more packages required by the build process.

```
apt-get install dosfstools libxrender1 libasound2 libatk1.0 libice6
```

Export a `SHELL` variable.

```
export SHELL=/bin/bash
```

Install unzip.

```
apt-get install unzip
```


Get your container id with:

```
sudo docker ps -a
```

Commit your changes.

```
sudo docker commit <container_id> ubuntu-firefoxos
```

Stop and remove the container.

```
sudo docker stop <container_id>
sudo docker rm <container_id>
```

Restart the container with a few more flags.

```
sudo docker run -t -i --privileged --expose 5037 -v /dev/bus/usb:/dev/bus/usb -v /host/data:/container/data ubuntu-firefoxos /bin/bash
```

The `--privileged --expose 5037 -v /dev/bus/usb:/dev/bus/usb` options are required for `adb` to be able to find your device.

Before building, make sure you enable the
[remote debugging mode](https://developer.mozilla.org/en-US/Firefox_OS/Debugging/Connecting_a_Firefox_OS_device_to_the_desktop)
on your phone.

> Open the Settings app, then Device Information > More Information > Developer.   
> In the developer menu, check "Remote debugging".



Then you are good to go:

```
cd /container/data/B2G
./config.sh flame
./build.sh
./flash.sh
```
