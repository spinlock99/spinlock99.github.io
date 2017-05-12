---
layout: post
title:  "The App Shell Model"
date:   2017-03-05 19:12:00 -0700
categories: pwa
---

## Cache what?

An *application shell* (or app shell) architecture is a great way to organize
your app so that it makes the most out of caching network requests.

The shell is the html, css and javascript that you need to boot your app. The
root of this shell is your `index.html`. When a user navigates to your app, they
only request the `index.html` page. All other network requests are kicked off by
it:

    <!DOCTYPE html>
    <html>
      <head>
        <link href="https://fonts.googleapis.com/css?family=Roboto:300,400,500" rel="stylesheet">
      </head>
      <body>
        <div id="container"></div>
        <script type="text/javascript" src="/bundle.js"></script>
      </body>
    </html>

In the example above, we load the roboto fonts in the page `<head>` and custom
javascript bundle in the `<body>`. In this example the `index.html`, `bundle.js`
and fonts make up our app shell. The first time a user visits our app, they will
need to download all of these assets before their app can start booting. Our
plan is to cache these assets so that the next time the user wants to user our
app, they can boot up the app immediately without waiting for the network. This
also means the user doesn't need a network to open our app in the future.

> Reliable Fast Performance

Repeast visits are extremely quick. Static assets and the UI (e.g. html,
javascript, images and css) are cached on the first visit so that they load
instantly on repeat visits. Content may be cached on the first visit but is
typically loaded when it is needed.

> Native Like Interactions

By adopting the app shell model, you can create experiences with instant,
native-application-like navigation and interactions, complete with offline
support.

> Economical Use of Data

Design for minimal data usage and be judicious in what you cache because listing
files that are non-essential (large images that are not shown on every page, for
instance) result in browsers downloading more data than is strictly necessary.
Even though data is relatively cheap in western countries, this is not the case
in emerging markets where connectivity is expensive and data is costly.
