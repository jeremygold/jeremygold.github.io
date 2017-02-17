---
layout: post
title: Raspberry PI Headless Setup for Headless operation
---

Just found a nice trick when setting up a Headless Raspberry PI to avoid scrambling around for an old keyboard, mouse, and HDMI cable, just to run raspi-config and enable ssh...

Just noticed on the [Raspberry PI ssh page](https://www.raspberrypi.org/documentation/remote-access/ssh/) that there's a better way to do this, by simply placing a file named ssh in the boot folder of your SD Card.

I'm on a Mac, so my general workflow for RPI setup is:

* Download latest Raspbian image from the [Raspbian Download Page](https://www.raspberrypi.org/downloads/raspbian/).
* I'm on a Mac, so I write this to an SD Card using [Apple PI Baker](https://www.tweaking4all.com/software/macosx-software/macosx-apple-pi-baker/) - _You'll need to scroll down a bit to find the download link_
* Once the write's done, the SD Card is normally automatically unmounted, so unplug and re-plug it...
* It should show up as a 'boot' device in Finder
* Open a terminal and:

```
$ cd /Volumes/boot
$ touch ssh
```

Then eject the SD Card and fire up the RPI. I normally use a wired connection the first time, and set up wifi over ssh later if needed.

To find the IP address of the RPI, I usually use [Fing](https://play.google.com/store/apps/details?id=com.overlook.android.fing&hl=en), or a bit of nmap magic (you'll need to update your address range depending on how your DHCP is set up):


```
$ sudo nmap -sn 192.168.1.0/24
```

