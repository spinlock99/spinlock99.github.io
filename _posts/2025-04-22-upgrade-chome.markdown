---
layout: post
title:  "Upgrade Chrome"
date:   2025-04-22 07:00:00 -0700
categories: ubuntu
---
{% include google_analytics.html %}

## Chrome is Out of Date

There was a pesky little note in the upper-righthand corner of chrome telling me
that it was out of date. But, `apt upgrade` wouldn't upgrade Chrome to the latest
version.

# How Chrome was Installed

I followed the instructions here:

> https://askubuntu.com/questions/1514599/how-do-i-install-google-chrome-on-ubuntu-24-04

Which added the following `sources.list`:

> /etc/apt/sources.list.d/google-chrome.sources

```
Enabled: no
Types: deb
URIs: https://dl.google.com/linux/chrome/deb/
Suites: stable
Components: main
Signed-By: -----BEGIN PGP PUBLIC KEY BLOCK-----
 .
 mQINBFcMjNMBEAC6Wr5QuLIFgz1V1EFPlg8ty2TsjQEl4VWftUAqWlMevJFWvYEx
 <-- snip -->
 M1aUAQDBwV7g9wPmcdRIjJS2MtK1JXHZCR1gVKb+EObct6RJOVw8s58ES5O9wGZm
 bVtIZ+JHTbuH+tg0EoRNcCbz
 -----END PGP PUBLIC KEY BLOCK-----
```

I'm not sure why the default install is disabled but just change the first line
to `Enabled: yes` and upgrade to get the latest versiion of Chrome.

```
spinlock@Derico:~$ sudo vi /etc/apt/sources.list.d/google-chrome.sources
spinlock@Derico:~$ sudo apt update
Hit:1 http://us.archive.ubuntu.com/ubuntu noble InRelease
Get:2 https://dl.google.com/linux/chrome/deb stable InRelease [1,825 B]
Get:3 http://us.archive.ubuntu.com/ubuntu noble-updates InRelease [126 kB]
Hit:4 http://security.ubuntu.com/ubuntu noble-security InRelease
Hit:5 http://us.archive.ubuntu.com/ubuntu noble-backports InRelease
Hit:6 https://ppa.launchpadcontent.net/keyd-team/ppa/ubuntu noble InRelease
Get:7 https://dl.google.com/linux/chrome/deb stable/main amd64 Packages [1,217 B]
Fetched 129 kB in 1s (129 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
2 packages can be upgraded. Run 'apt list --upgradable' to see them.
N: Skipping acquire of configured file 'main/binary-i386/Packages' as repository 'https://dl.google.com/linux/chrome/deb stable InRelease' doesn't support architecture 'i386'
spinlock@Derico:~$ apt list --upgradable
Listing... Done
google-chrome-stable/stable 135.0.7049.95-1 amd64 [upgradable from: 131.0.6778.139-1]
ubuntu-drivers-common/noble-updates 1:0.9.7.6ubuntu3.2 amd64 [upgradable from: 1:0.9.7.6ubuntu3.1]
spinlock@Derico:~$ sudo apt upgrade
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Calculating upgrade... Done
The following package was automatically installed and is no longer required:
  python3-netifaces
Use 'sudo apt autoremove' to remove it.
Get more security updates through Ubuntu Pro with 'esm-apps' enabled:
  graphicsmagick libzvbi-common libcjson1 libavcodec60 libzvbi0t64
  libgraphicsmagick++-q16-12t64 libavutil58 libswresample4 libavformat60
  libgraphicsmagick-q16-3t64
Learn more about Ubuntu Pro at https://ubuntu.com/pro
The following upgrades have been deferred due to phasing:
  ubuntu-drivers-common
The following packages will be upgraded:
  google-chrome-stable
1 upgraded, 0 newly installed, 0 to remove and 1 not upgraded.
Need to get 115 MB of archives.
After this operation, 11.2 MB of additional disk space will be used.
Do you want to continue? [Y/n]
Get:1 https://dl.google.com/linux/chrome/deb stable/main amd64 google-chrome-stable amd64 135.0.7049.95-1 [115 MB]
Fetched 115 MB in 5s (22.4 MB/s)
(Reading database ... 233192 files and directories currently installed.)
Preparing to unpack .../google-chrome-stable_135.0.7049.95-1_amd64.deb ...
Unpacking google-chrome-stable (135.0.7049.95-1) over (131.0.6778.139-1) ...
Setting up google-chrome-stable (135.0.7049.95-1) ...
Processing triggers for gnome-menus (3.36.0-1.1ubuntu3) ...
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for desktop-file-utils (0.27-2build1) ...
```
