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


```
https://github.com/lwfinger/rtw89
```
