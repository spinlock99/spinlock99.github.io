---
layout: post
title:  "repository 'https://dl.google.com/linux/chrome/deb stable InRelease' doesn't support architecture 'i386'"
date:   2025-05-09 07:00:00 -0700
categories: ubuntu
---
{% include google_analytics.html %}

## APT Update Complains about Chrome not Supporting i386

```
spinlock@Derico:~$ sudo apt update
[sudo] password for spinlock: 
Hit:1 http://us.archive.ubuntu.com/ubuntu noble InRelease
Hit:2 http://us.archive.ubuntu.com/ubuntu noble-updates InRelease                                      
Get:3 http://security.ubuntu.com/ubuntu noble-security InRelease [126 kB]                                                    
Hit:4 http://us.archive.ubuntu.com/ubuntu noble-backports InRelease                                                                
Hit:5 https://dl.google.com/linux/chrome/deb stable InRelease                                                                      
Hit:6 https://ppa.launchpadcontent.net/keyd-team/ppa/ubuntu noble InRelease                                       
Get:7 http://security.ubuntu.com/ubuntu noble-security/main amd64 Components [21.5 kB]
Get:8 http://security.ubuntu.com/ubuntu noble-security/restricted amd64 Components [212 B]
Get:9 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Components [52.3 kB]
Get:10 http://security.ubuntu.com/ubuntu noble-security/multiverse amd64 Components [212 B]
Fetched 200 kB in 1s (156 kB/s)     
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
14 packages can be upgraded. Run 'apt list --upgradable' to see them.
N: Skipping acquire of configured file 'main/binary-i386/Packages' as repository 'https://dl.google.com/linux/chrome/deb stable InRelease' doesn't support architecture 'i386'
```

# Root Cause

Following along here:

> https://askubuntu.com/questions/741410/skipping-acquire-of-configured-file-main-binary-i386-packages-as-repository-x

Google droped support for 32-bit Chrome so `apt` is complaining because i386
is configured as a foreign architecture.

# Solution

Because my sources are in the DEB822 style format, I needed to add an
architectures line to my sources file for chrome:

```
spinlock@Derico:~/src/spinlock99.github.io/_posts$ sudo cat /etc/apt/sources.list.d/google-chrome.sources
Enabled: yes
Types: deb
URIs: https://dl.google.com/linux/chrome/deb/
Suites: stable
Components: main
Architectures: amd64
...
```

```
spinlock@Derico:~/src/spinlock99.github.io/_posts$ sudo apt update
Hit:1 http://security.ubuntu.com/ubuntu noble-security InRelease
Hit:2 http://us.archive.ubuntu.com/ubuntu noble InRelease
Hit:3 http://us.archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:4 https://dl.google.com/linux/chrome/deb stable InRelease
Hit:5 http://us.archive.ubuntu.com/ubuntu noble-backports InRelease
Hit:6 https://ppa.launchpadcontent.net/keyd-team/ppa/ubuntu noble InRelease
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
4 packages can be upgraded. Run 'apt list --upgradable' to see them.
```
