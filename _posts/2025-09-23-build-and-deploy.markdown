---
layout: post
title:  "Build and Deploy"
date:   2025-09-23 07:00:00 -0700
categories: debian
---
{% include google_analytics.html %}

# Build

Unlike Distilery, Mix Releases aren't already tarred and gzipped. So, we can
simplify the port of Bootleg to use Mix Releases by creating a tar archive that
can take it's place in the Deploy process.

> mix bootleg.build

# Deploy

> mix bootleg.deploy

# Next Steps

* Get it running with SystemD.
* Put it behind Nginx.
* Self Signed Cert.
