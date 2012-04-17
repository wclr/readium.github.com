---
layout: post
title: "Running Readium on Mobile"
date: 2012-04-16 17:14
comments: true
categories: 
---

Lately we have been experimenting with running the Readium source code in new and interesting ways. The entire application is written in HTML5 (html, css and javascript) so theoretically we should be able to package it up like a web site and run it in any modern browser. Practically we are utilizing a few cutting edge HTML5 features for which there is not yet widespread support (namely CSS3 regions and the HTML5 filesystem api). That said we do have a working, albeit somewhat fragile demo, that you can check out [here](http://github.readium.org). It is still riddled with bugs so if you get into trouble, refresh liberally or try another browser (for those of you with android devices, the demo works surprisingly well on [Chrome for Android](https://play.google.com/store/apps/details?id=com.android.chrome&hl=en)).

##How it works

Because we cannot access the HTML5 filesystem API in most browsers, we are hosting the EPUB files on the server and fetching the content via AJAX from the viewer when they are opened. This means that you cannot actually load an EPUB file into this version of Readium but we think this will be OK for a lot of use cases (think online bookstore that wants to allow access to files for its customers) and there may be ways to implement this functionality (see below).

The next thing you might notice is that pagination of re-flowable content is very broken on most browsers. That is because almost nobody has support for the CSS regions property yet. Thankfully one of contributors has come up with a polyfill that should provide a fallback in these cases and [is currently working on integrating it](https://github.com/readium/readium/issues/61).

##What's next

There is obviously a lot left to do if we want to get this version to a publicly usable state. Here are a few things we have thought about taking on:

- __Fix bugs__ -  there are a lot of issues that are stemming from the fact that serving thing over the web is different than serving them from a .crx. We need to take care of these one by one
- __Tune up the UI / UX__ - the Readium UI needs to be tweaked a little to make if feel at home on a mobile touch interface.
- __Package up the build as a Phonegap application__ - This may be a little further down the road, but once done it will give as another means to interact with the filesystem and let users import and read their own EPUB3 files on their devices.

As always in anyone thinks they can help with any of these items (or anything else for that matter). Please don't hesitate to create items in the [github issue tracker](https://github.com/readium/readium/issues?sort=created&direction=desc&state=open) or better yet submit pull requests with patches.