---
layout: post
title:  "Installing WebApps on SmartPhones"
date:   2017-03-05 19:12:00 -0700
categories: pwa
---
{% include google_analytics.html %}

## Getting on the Homescreen

My favorite prank to pull on silicon valley web developers is to open an app I'm working on by tapping an icon on my iPhone and showing off the latest version. They'll say things like, "That's cool. I haven't published anything to the app store yet." They're blown away when I tell them, "neither have I; it's a web app."

Of course, having your app live on the user's homescreen has benefits beyond trolling. Reengagement can be orders of magnitude greater for apps that live on the homescreen. Bloomberg has six times the monthly users on its mobile website than they do on their native app. But, the app generates 25 times the page views. In this post, I'll show you how to install your web app on the homescreen.

# iOS

We're going to be a bit restricted in the platforms we can support. For iOS, we'll only be able to support safari. This is actually the most common mobile web setup in north america so it's worth some extra effort to support it.

> Specifying a Webpage Icon for Web Clip

Apple calls the link to your app a "web clip" when it's represented by an icon on the homescreen. To specify an icon to represent your web app you'll need to:

* specify an icon for the entire website by adding a png to the root document folder called "apple-touch-icon.png"
* specify an icon for a single page by adding a link:

  `<link rel="apple-touch-icon" href="/custom-icon.png">`
* specify multiple icons for different device resolutions:

  ```
  <link rel="apple-touch-icon" href="touch-icon-iphone.png">
  <link rel="apple-touch-icon" sizes="152x152" href="touch-icon-ipad.png">
  <link rel="apple-touch-icon" sizes="167x167" href="touch-icon-ipad-retina.png">
  <link rel="apple-touch-icon" sizes="180x180" href="touch-icon-iphone-retina.png">
  ```

> Specifying a Launch Screen Image

On iOS, similar to native applications, you can specify a launch screen image that is displayed while your web application launches. This is especially useful when your web app is offline. By default, a screenshot of the last time the app was used is shown. To set another startup image, add a link element to the page:

  `<link rel="apple-touch-startup-image" href="/launch.png">`

> Adding a Launch Icon Title

On iOS, you can specify a web application title for the launch icon. By default, the `<title>` tag is used. To set a different title, adda meta tag to the page:

  `<meta name="apple-mobile-web-app-title" content="AppTitle">`

> Hiding Safari User Interface Components

On iOS, as part of optimizing your app, have it use the standalone mode to look more like a native application. When you use this standalone mode, Safari is not used to display web content -- specifically, there is no browser url text field at the top of the screen or button bar at the bottom of the screen. Only a status bar appears at the top of the screen.

Set teh `apple-mobile-web-app-capable` meta tag to yes to turn on standalone mode:

  `<meta name="apple-mobile-web-app-capable" content="yes">`

> Changing the Status Bar Appearance

If your web application displays in standalone mode like that of a native application, you can minimize the status bar that is displayed at the top of the screen on iOS. Do so using the `status-bar-style` meta tag.

This meta tag has no effect unless you first specify stanalone mode as described above. Then use the status bar style meta tag, `apple-mobile-web-app-status-bar-style`, to change the appearance of the status bar depending on your application needs. For example, if you want the entire screen, set the status bar style to translucent black.

`<meta name="apple-mobile-web-app-status-bar-style" content="black">`

> Linking to Other Native Apps

Your web app can link to other build-in iOS apps by creating a link with a special url. Available functionality includes calling a phone number, sending an sms or iMessage, and opening youtube videos in the native app if it is installed. For example, to link a phone number, structure an anchor element like this:

`<a href="tel:1-408-555-5555">Call me</a>`

# Android

Chrome on android uses a totally different mechanism for installation called the "web app manifest." The web app manifest is a simple json file that gives you the ability to control how your app appears on the user's homescreen.

Web app manifests provide the ability to save a site bookmark to a device's homescreen. When the site is launched this way:

* it has a unique icon and name so that users can distinguish it from other sites.
* it displays something to the user while resources are downloaded or restored from cache.
* it provides default display characteristics to the browser to avoid too abrupt transition when site resources become available.

It does all this through the simple mechanism of metadata in a text file. That's the web app manifest.

Let's start by looking at a simple web app manifest:

```
// manifest.json
{
  "short_name": "TestApp",
  "name": "My Test App",
  "icons: [
    {
      "src": "launcher-icon-1x.png",
      "type": "image/png",
      "sizes": "48x48"
    },
    {
      "src": "launcher-icon-2x.png",
      "type": "image/png",
      "sizes": "96x96"
    },
    {
      "src": "launcher-icon-4x.png",
      "type": "image/png",
      "sizes": "192x192"
    }
  ]
}
```

> Tell the Browser About Your Manifest

When you have created the manifest and it's on your site, add a `link` tag to all of the pages on your web app:

`<link rel="manifest" href="/manifest.json">`

>Set a Start URL

If you don't provide a `start_url`, the current page is used. This is often a reasonable default but, by explicitly defining a `start_url`, we can add a query string -- for example -- that will let us monitor how much of our traffic comes from users launching our app from their homescreen vs navigating to our app with their browser:

`"start_url": "/?utm_source=homescreen"`

This can be anything you want but the value above has the advantage of being meaningful to google analytics.

> Customize the Icons

When a user adds your site to their homescreen, you can define a set of icons for the browser to use:

```
"icons": [
  {
    "src": "images/touch/icon-128x128.png",
    "type": "image/png",
    "sizes": "128x128"
  },
  {
    "src": "images/touch/apple-touch-icon.png",
    "type": "image/png",
    "sizes": "152x152"
  },
  {
    "src": "images/touch/ms-touch-icon-144x144-precomposed.png",
    "type": "image/png",
    "sizes": "144x144"
  },
  {
    "src": "images/touch/chrome-touch-icon-192x192.png",
    "type": "image/png",
    "sizes": "192x192"
  }
]
```

> Add a Splash Screen

When you launch your app from the homescreen three things happen behind the scenes:
1. chrome launches
2. the renderer that displays the page starts up
3. your site loads from the network (or cache if you're using service workers)

While this is happening, the screen goes white and appears to be stalled. This is especially apparent on a flaky network where your app might take a second or two to become visible on the homepage.

To provide a better user experience, you can replace the white screen with a title, color, and images.

> Set an Image and Title

A splash screen image is drawn from the `icons` array in the manifest file. Chrome chooses the image that is closest to 128dp for the device. The title is pulled from the `name` field.

> Set the Background Color

Specify background color using the appropriately named `background_color` property. Chrome uses this color from the time your app is launched until first render.

`"background_color": "#2196F3"`

It's a good idea to use the same background color for the splash screen as your apps load page. This way, you'll have a seamless transition from launching to booting up your app.

> Set a Theme Color

Use the `theme_color` property to set the color of the toolbar. It's also recommended to match this color to the `theme-color` `meta` tag.

> Set the Launch Style

To make your app hide chrome's UI, add this to your manifest:

`"display": "standalone"`
