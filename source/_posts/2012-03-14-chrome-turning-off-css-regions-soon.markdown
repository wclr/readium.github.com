---
layout: post
title: "Chrome Disabling CSS3 Regions Soon"
date: 2012-03-14 17:17
comments: true
author: Matthew Robertson
categories: 
---
Way back in version 16 the Chrome browser began shipping support for [CSS3 regions](http://dev.w3.org/csswg/css3-regions/). This feature defined a declarative markup to have HTML content flow into multiple regions such that the overflow from the first regions will spill over into the second region, overflow from the second will flow into the third etc. Parts of the implementation were shaky and / or missing (eg the javascript APIs were completely stubbed out) nevertheless, it was supported and it worked in the basic use cases, so we decided to leverage it in Readium to perform pagination of reflowable content. 

Our regions based pagination algorithm basically looks like this:

* `whenever a repagination event occurs:`
  * `make a pessimistic guess about how many pages are needed`
  * `add that many pages to the dom`
  * `while the last page has overflow:`
    * `add another page` 

This was all working well enough in terms of paginating the content. There were some bugs related to click-able areas not being rendered where they were expected but we assumed these were bugs in Webkit / chrome and would get ironed out in future versions. Then one day I found out that regions were going to be [shut off by default in webkit](https://bugs.webkit.org/show_bug.cgi?id=78525#c0). Sure enough I installed [chrome canary](http://tools.google.com/dlpage/chromesxs), regions were gone and Readium did not handle it well. This was not good news for us.

{% img center http://fuuu.us/152.png %}

## Challenge Accepted

We decided that a short term solution to our problem would be to find a way to enable CSS3 regions to be turned back on. With a little support from the awesome chromium team at google, I managed to stumble through the chromium source code, submit [a patch](https://chromiumcodereview.appspot.com/9523002/) that allows regions to be turned on via the command line. This patch has landed.

## So What Can You do About it?

If you are using chrome Canary or you are from the future where chrome has moved to a version in which CSS3 regions are off by default here are the steps you can take to turn them back on:

1. Open a new tab in your browser and enter `about:flags` into the url bar
2. In the page that opens find an entry called `Enable CSS Regions` and click the **enable** link
3. Restart chrome for the changes to take affect

## A Better Solution

We realize that the above solution does not provide the best user experience, and we are currently seeking other solutions. One idea is to distribute Readium with a custom build of Webkit that has regions turned on. This may seem a bit heavy handed, but we were planning on creating a custom Webkit in order support vertical typography all along. A simpler solution that is likely to show up soon will be to degrade gracefully when regions are not available or come up with a pagination strategy that works without CSS regions. If you have any good ideas for how we might do this please put them in the [issue tracker](https://github.com/readium/readium/issues?sort=created&direction=desc&state=open).
