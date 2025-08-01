---
layout: post
title:  "New Laptop: HP ProBook"
date:   2025-07-22 07:00:00 -0700
categories: debian
---
{% include google_analytics.html %}

## Install Debian

I'm missing most of the details because it's taken a couple of days to install
Debian on my new laptop. The major hurdle was getting the WIFI up and running.
This is the one area where Debian was less friendly than Ubuntu. I think^(TM)
that the issue is with non-free drivers for the Realtek device that supports
Bluetooth and WIFI.

The Debian 12 install image is very minimal so it was much easier to run the
network installer over ethernet to get everything up and running. Then, work on
the WIFI with a more usable system.


## WIFI

I found the following kernel modules to support my hardware. I forget how I
determined which chipset I was running but I think it was just googleing the
HP ProBook and Debian.

```
https://github.com/lwfinger/rtw89
```

### Setup

Install the necessary tools to build the kernel module...

```
sudo apt-get update
sudo apt-get install make gcc linux-headers-$(uname -r) build-essential git
```

### Build

```
git clone https://github.com/lwfinger/rtw89.git
cd rtw89
make
sudo make install
```

### DKMS Packaging

This step creates a debian package for the newly built kernel module so that
the device drivers are managed by `apt`. This was a huge help in getting the
WIFI to work because Debian Stable (i.e. 12 as of this writing) used an old
Linux Kernel that the modules did not work with. I upgraded the entire system
to Debian Testing (i.e. Debian 13 - codename Trixie) rather than only upgrading
the Kernel and that was a good call because I really like Trixie and it's been
stable so far.

```
sudo apt install dh-sequence-dkms debhelper build-essential devscripts git-build-recipe
```

```
# If you've already built as above clean up your workspace or check one out specially (otherwise some temp files can end up in your package)
git clean -xfd

git deborig HEAD
dpkg-buildpackage -us -uc
sudo apt install ../rtw89-dkms_1.0.2-3_all.deb
```

## Upgrade to Debian 13 - Trixie

The kernel modules didn't work so I decided to upgrade the sytem to testing
because it uses Linux version 6.12 which should work with the modules.

### Update Sources List

First, we edit `/etc/apt/sources.list` and replace `bookworm` with `trixie`s:

```
#deb cdrom:[Debian GNU/Linux 12.11.0 _Bookworm_ - Official amd64 NETINST with firmware 20250517-09:51]/ trixie contrib main non-free-firmware

deb http://deb.debian.org/debian/ trixie main non-free-firmware
deb-src http://deb.debian.org/debian/ trixie main non-free-firmware

deb http://security.debian.org/debian-security trixie-security main non-free-firmware
deb-src http://security.debian.org/debian-security trixie-security main non-free-firmware

# trixie-updates, to get updates before a point release is made;
# see https://www.debian.org/doc/manuals/debian-reference/ch02.en.html#_updates_and_backports
deb http://deb.debian.org/debian/ trixie-updates main non-free-firmware
deb-src http://deb.debian.org/debian/ trixie-updates main non-free-firmware

# This system was installed using small removable media
# (e.g. netinst, live or single CD). The matching "deb cdrom"
# entries were disabled at the end of the installation process.
# For information about how to configure apt package sources,
# see the sources.list(5) manual.
```

### Upgrade the System

Given that this was a clean install, I was pretty agressive with the distribution
upgrade because I knew it would be easy to start over from the USB.

```
apt update
apt upgrade
apt full-upgrde
```

And ... everything went great. The sytem booted right up and the WIFI is working
great. I should go back and do a clean install to make sure I needed the Realtek
drivers and that Trixie doesn't just work right out of the box.
