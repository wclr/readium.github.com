---
layout: post
title: Readium Turns 0.4
date: 2012-06-21 10:17
comments: true
author: Justin Hume
categories: 
---

We recently pushed out version 0.4.0 of Readium so it is time for an update.

## Contributing

First up, there has been a lot of recent interest from people who want to contribute to the Readium project, which is always encouraging and very welcomed. It's especially helpful to have contributors who can read and debug the rendering of ePubs in languages other than English. Matthew did a great job (more below) getting [vertical Japanese text](http://code.google.com/p/epub-samples/wiki/SamplesListing#kusamakura-japanese-vertical-writing) to render, which he was pretty excited about. Did it actually render something that made sense? We had no idea (it did). But it _was_ vertical. 

We also have users contributing bug reports, which is definitely helpful as these improve our understanding of what kind of ePubs are getting put together out there. So, if you've come across the Readium project and tried your unique ePub and it's not working, let us know by creating an issue in the [tracker](https://github.com/readium/readium/issues?direction=desc&sort=created&state=open). We can't promise quick fixes for everything, but it goes a long way to helping build a full-spec reader. 

## docco

We had a suggestion from [Simon Schmid](https://github.com/schmidsi) to set up [docco](http://jashkenas.github.com/docco/) to make the code base more accessible and understandable. docco pulls comments directly out of source files and lines them up with your code in a nicely formatted web page. It uses markdown and only pulls out the // comments; /**/ comments are left in the source. In any case, you don't have to do anything but comment, we'll take care of deploying the updated docco-mentation. We've got it set up and it can be accessed from the [documentation](http://github.readium.org/docs/ebook.html) tab on this blog (you can navigate to other docco pages with the widget in the top right). In general contributing documentation is a great way to help out while familiarizing yourself with an open source project and Readium could use some more comments, so comment away. 

## IRC channel 

We've set up an IRC channel for the Readium project on freenode.net. The channel is #Readium. Matthew or I will try to be on there during PST working hours and hopefully we can answer questions if they come up. I can't imagine we're going to have too many flame wars about Readium but since this _is_ the internet, I would feel negligent not to remind everyone to keep it respectful. 

## CSS regions no more

In order to better support different forms of writing - right-to-left, vertical etc. - Matthew changed the reflowable pagination approach from the use of CSS3 regions to CSS columns. This was a big step forward for Readium as CSS columns are supported on [almost all modern browsers](http://caniuse.com/#feat=multicolumn). Also, because this approach is able to execute entirely within an `iframe` it solved a number of other issues we had been having (eg scoping of style rules). We are also able to allow spine level scripting for the reflowable content, which a lot of EPUB content creators were really excited to start working with.

## Alternate Style Tags

An implementation of the Alternate Style Tags specification has been added to Readium. It is set up to interpret arbitrary sets of tags generally, although currently only day/night tags are specifically handled. This can be demonstrated with the use of the [Wasteland](http://code.google.com/p/epub-samples/wiki/SamplesListing#wasteland) ePub from the Sample-Project, which has an alternate style set for "night" mode. If anyone sees any problems, missing parts of the spec or misinterpretations, please let me know! 

## Wrap-up

So that's it for now. As always if you have questions, want to contribute, or submit bug reports, we can be contacted on [github](https://github.com/readium/readium/), by email, or now on IRC!  