---
layout: post
title:  "Remapping CAPSLOCK in Ubuntu"
date:   2024-10-09 10:45:00 -0700
categories: os
---
{% include google_analytics.html %}

## Remapping CAPSLOCK in Ubuntu

> indexed

Make sure that your site can be crawled by google and bing. You can use
[fetch as google](https://support.google.com/webmasters/answer/6066468) to see
how their webcrawler interprets your page.

> social

Leverage facebook and twitter to make your app [socially discoverable](https://developers.google.com/web/fundamentals/discovery-and-monetization/social-discovery/).

> use the history api

For single page apps, ensure the site doesn't use fragment identifiers. For
example everything after the `#!` in `https://example.com/#!user/26601`.

> don't fuc the page

If your page jumps when your load it your have a flash of unformatted content
(fuc'd) page. make sure that images and ads have fixed sizing in css or inline
the element.

> remember scroll postion

When the user clicks the back button, they expect to see the page as it was when
they clicked off of it. Store scroll position and any other page state in the
history api so that you can restore it when the user clicks back to the page. A
good routing library should handle this for you.

> inputs aren't hidden by the keyboard

Use features like `Element.scrollIntoView()` and `Element.scrollIntoViewIfNeeded()`
to prevent the keyboard from covering input elements.

> cache-first networking

Use service workers to serve data from the cache and then pull it from the network.

> push notifications

* provide context to the user about hos notifications will be used
* encourage users to turn on push notifications (but not too aggressively)
* dim the screen when requesting permission
* a good push notification is timely, precise and relevant
* make it easy to enable and disable notifications

> single sign on across devices

Let your users sign in with their social network credentials.

> payments

Now that our app is baller lets make that money.
