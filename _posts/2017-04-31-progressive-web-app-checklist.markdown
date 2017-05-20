---
layout: post
title:  "Progressive Web App Checklist"
date:   2017-04-31 21:13:00 -0700
categories: pwa
---
{% include google_analytics.html %}

## Progressive Web App Checklist

Progressive Web Apps (PWA) are reliable, fast, and engaging, although there are
many things that can take a pwa from baseline to exemplary experience.

To help teams create the best possible experiences we've put together this
checklist which breaks down all the things we think it takes to be a baseline
pwa by providing a more meaningful offline experience, reaching interactive even
faster and taking care of many more important details.

> site is served over https

If you don't have https yet, checkout letsencrypt.org to get started.

> pages are responsive on tables & mobile devices

The best place to start is with a css grid. From there, you can move on to more
advanced techniques like media queries. If you're using a frontend framework
like React you can optionally render components for specific screen sizes.

> offline support

The start url (at least) should load without a network connection. You can use
service workers to cache your assets on Chrome but you'll need to fall back to
App Cache to support Safari. Apple has service worker support on their 5 year
plan so hopefully we can settle on a single solution soon.

> install to the homescreen

Again, we'll need to handle this differently for Safari and Chrome. Safari has
you add a link to the icon you'd like to [install on the homescreen](https://developer.apple.com/library/content/documentation/AppleApplications/Reference/SafariWebContent/ConfiguringWebApplications/ConfiguringWebApplications.html):

```<link rel="apple-touch-icon" href="/custom_icon.png">```

While Chrome has you add a [web app manifest](https://developers.google.com/web/fundamentals/engage-and-retain/web-app-manifest/):

```<link rel=manifest" href="/manifest.json">```

> cross browser support

We've talked about Chrome and Safari. But you don't want to forget Firefox and
even Edge.

> smoot page transitions

Give the user immediate feedback that their navigation did something. Pay
attention to loading states. You can use a [skeleton screen](http://hannahatkin.com/blog/tag/skeleton-screens/)
to show the user any information you have while the reset is loaded on demand.

> deep linking

Each page of your app should have a url that will route the user to it. Your
users can then share links to specific pages of your app simply by sharing a
url. You should also consider routing modals and other configurations your page
can have. A user might want to bookmark a url that opes an app with the sidebar
minimized for instance.
