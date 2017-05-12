---
layout: post
title:  "Dealing with Flaky Networks"
date:   2017-03-05 19:12:00 -0700
categories: pwa
---

## When the Network Goes Down, We Stay Up

The two big differences in creating apps for phones vs the desktop are screen
size and network reliability. When your serving a page to a desktop user, you
expect their network connection to be consistent. If they start with wifi they
aren't going to switch to dial-up in the middle of their session. Mobile users
change their network all the time. An average commuter will hit 3 to 5 cellular
towers on their ride in to work. They might even loose connectivity altogether
by going through a tunnel.

We deal with flaky networks by caching all of the assets we need to run our app.
This let's us boot up our app even when it's offline. The two technologies we'll
look at for doing this are: app cache and service workers. App cache is an older
spec that wasn't implemented consistently across browsers. This meant you'd
never really know if you had busted your cache or if you were still serving old
assets. Service workers are the next generation that have solved these problems
and added features like push notifications and background sync. We're going to
cover app cache because safari does not support service workers yet (though it
is on their 5 year plan).

# App Cache

html5 privides an application caching mechanism that lets web apps run offline.
In order to enable app cache, you must include it in your `html` page:

    <html manifest="example.appcache">
    ...
    </html>

The manifest attribute references a cache manifest file, which is a text file
that lists resources (files) that the browser should cache for your application.

You should include the manifest attribute on every page of your application that
you want cached. The browser does not cache pages that do not contain the
manifest attribute, unless suck pages are explicitly listed in the manifest file
itself. You do not need to list all the pages you want cached in the manifest
file, the browser implicitly adds every page that the user visits and that has
the manifest attribute set to the application cache.

# Service Workers

A service worker is a script that your browser runs in the background to handle features that don't need a webpage or user interaction. Running in the background means that they will not block the ui even if they have to run a very long computation. Today, they already include features like [push notifications](https://developers.google.com/web/updates/2015/03/push-notifications-on-the-open-web) and [background sync](https://developers.google.com/web/updates/2015/12/background-sync). In the future, service workers will support other things like periodic sync or geofencing. The core feature we'll be interested in is their ability to intercept and handle network requests and manage a cache of responses.

> It's a [javascript worker](https://www.html5rocks.com/en/tutorials/workers/basics/)

This means it can't access the dom directly. Service Workers communicate through
the [postMessage](https://html.spec.whatwg.org/multipage/workers.html#dom-worker-postmessage)
interface.

> It's a programmable network proxy

It gives control over how network requests from your page are handled.

> It runs intermittently

Your service worker will be terminated when it's not in use, and restarted when
it's needed next. This saves resources but it means you can't rely on global
state in your `onfetch` and `onmessage` handlers. Service workers do have access
to indexedDB so you can use that for persisting information across restarts.

> They use promises extensively

Service workers are asynchronous and provide a promise based api for dealing
with concurrency.

### The Service Worker Life Cycle

A service worker has a lifecyle that is completely separate from your web page.

```
                               .-----------------.
Installing --> Error           v                 |
            |-> Activated -> Idle -> Terminated -|
                          |-> Fetch/Message -----/
```

To install a service worker for your site, you need to register it, which you do
in your page's javascript. Registering a service worker will cause the browser
to start the service worker install step in the background.

Typically during the install step, you'll want to cache some static assets. If
all the files are cached successfully, then the service worker becomes
installed. If any of the files fail to download and cache, then the install step
will fail and the service worker won't activate (i.e. won't be installed). If
that happens, don't worry, it'll try again next time. But that means if it does
install, you know you've got those static assets in the cache.

After the activation step, the service worker will control all pages that fall
under its scope, though the page that registered the service worker for the
first time won't be controlled until it's loaded again. Once a service worker is
in control, it will be handle fetch and message events that occur when a network
request is made from your page.

> Register a Service Worker

Before installing a service worker you need to bootstrap the process by
registering it in your page. This tells the browser where this javascript file
defining your service work lives.

    if("serviceWorker" in navigator) {
      window.addEventListener("load", () => {
        navigator.serviceWorker.register("/sw.js").then(
            success => console.log("service worker registered successfully", success),
            error => console.log("service worker registration failed", error)
        );
      });
    }

The registration logic for your service worker lives in the page but the
`install` event is handled by the service worker itself.

    var CACHE_NAME = "v1";
    var urlsToCache = [
      "/",
      "/styles/main.css",
      "/script/main.js"
      ];

    self.addEventListener("install", event => {
      event.waitUntil(
        caches.open(CACHE_NAME).then(cache => {
          console.log("opened cache");
          return cache.addAll(urlsToCache);
        })
      )
    });

If all the files are successfully cached, then the service worker will be
installed. If *any* of the files fail to download, then the install step will
fail. This allows you to rely on having all the assets that you defined, but
does mean you need to be careful with the list of files you decide to cache in
the install step. Defining a long list of files will increase the chance that
one file may fail to cache, leading to your service worker not being installed.

> Cache and Return Requests

Now that you've installed a service worker, you probably want to return one of
your cached responses, right?

After a service worker is installed and the user navigates to a different page
or refreshes, the service worker will begin to receive `fetch` events:

    self.addEventListener("fetch", event => {
      event.responseWith(
        caches.match(event.request).then(response => {
          if (response) { return response };
          return fetch(event.request);
        })
      );
    });

Here we've defined our `fetch` event and within `event.respondWith()`, we pass
in a promise from `caches.match()`. This method looks at the request and finds
any cached results from any of the caches your service worker created.
