---
layout: post
title:  "Build and Deploy"
date:   2025-09-23 07:00:00 -0700
categories: debian
---
{% include google_analytics.html %}

# Build

I've changed directions since last time. Unlike Distilery, Mix Releases aren't
already tarred and gzipped. I had just been copying the entire release directory
but this caused problems in the Deploy phase. Instead of rewriting all of the
deploy code, we can simplify the port of Bootleg to use Mix Releases by creating
a tar archive that can take it's place in the Deploy process.

> config/deploy.exs

> ../bootleg/lib/bootleg/tasks/deploy.exs

> mix bootleg.build

# Deploy

> config/deploy/production.exs

> ../bootleg/lib/bootleg/tasks/build/remote.exs

> mix bootleg.deploy

# Next Steps

* Get it running with SystemD.
* Put it behind Nginx.
* Self Signed Cert.
