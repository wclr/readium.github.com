---
layout: post
title: "Unpacking EPUBs in Readium"
date: 2012-05-04 19:29
comments: true
author: Matthew Robertson
categories: 
---

Something has been weighing heavily on my conscience for a while. Readium suffers from a serious lack of technical documentation. The closest I have come to putting anything into writing was creating [this wiki page](https://github.com/readium/readium/wiki/Architecture) over two months ago. If we want to attract contributors to the project, we need to make diving into the source code as streamlined and painless as possible. Our architecture wiki is a pretty pathetic attempt in this regard. 

But today I am going to take the first step towards writing this wrong. I am planning a series of (at least) 3 blog posts explaining how the internals of Readium operate. I think the end result will be interesting not only to Readium contributors, but also to ebook publishers, developers of other ebook reading systems and anyone building large applications with HTML5 / javascript.

## Unpacking an EPUB

I felt that the most natural place to start diving into Readium is the beginning. Readium currently ships with an empty library, so the first thing that every new user of our system does is import a file into their library.

### How do you wanna do this?

As of writing, Readium offers users three ways to import content into their library: from a remote URL, from their local filesystem in packed format and from their local filesystem as an unpacked directory. Of these three cases, importing from the local filesystem in packed form is the most complex (the other two options are really just subcases of this problem), so we will explore this use case for the rest of the post.

### Pick a file any file.

To enable a user to select a file from their local filesystem Readium makes use of a plain html `<input type='file'>` element. But if you have done any web development before you will notice that what happens next is completely non-traditional. In a classic web application an HTML form is really a user friendly tool for adding parameters to an HTTP request. When you click the __submit__ button of an HTML form the browser serializes whatever you typed in the form's fields into `application/x-www-form-urlencoded` data, tacks it on as the payload of an HTTP POST request (usually) and sends to the server. 

But Readium is 100% client side; there is no server to send the data to. We use the `<input type='file'>` element only to get a copy of users file into our javascript sandbox. Once we have that we use the HTML5 javascript method [window.URL.createObjectURL](https://developer.mozilla.org/en/DOM/window.URL.createObjectURL) to create a URL that points to the file they selected. This is the exactly where we would start if instead of selecting an EPUB from their filesystem, the user entered a URL (like I said, subcases).

### XYZ PDQ

The next step is to unpack the file. EPUBs use the [ZIP file format](http://en.wikipedia.org/wiki/Zip_(file_format)) to compress and archive thier contents. To decompress this content we make use of an [open source library](http://cheeso.members.winisp.net/examples.aspx#jsunzip) for unzipping the content in javascript. Unfortunately, decompressing files in javascript is dreadfully slow. This is because eventhough it has them, the [bitwise operators](https://developer.mozilla.org/en/JavaScript/Reference/Operators/Bitwise_Operators) in javascript are slow. This is because javascript does not have an `int` type. But, when performing bitwise operations it pretends that it does. The result is that in order for a javascript engine to perform a bitwise operation, it needs to do a conversion from its floating point `Number` type to the equivalent 32bit signed integer, perform the operation, and then convert back to its `Number` type. 

This performance constraint is what motivated us to start allowing users load an unpacked directory into Readium. To get this working we use the nifty `webkitdirectory` attribute on the form `<input>`. If you have never seen this in action before (or never knew of its existence) there is a pretty good demo available [here](http://html5-demos.appspot.com/static/html5storage/demos/upload_directory/index.html) or better yet, check it out in the Readium [source code](https://github.com/readium/readium).

### The Extraction

All of Readium's logic for importing an epub from both zipped and unpacked formats is contained in [/scripts/extractBook.js](https://github.com/readium/readium/blob/master/scripts/extractBook.js). For those of you who speak the [Unified Modeling Language](http://en.wikipedia.org/wiki/Unified_Modeling_Language), here is a rundown of what is in there:

{% img center https://github.com/readium/readium/raw/master/docs/unpacking_uml.png %}

Perched at the top of the hierarchy is `Backbone.Model`. This is not actually defined in Readium, it is an object provided by [Backbone.js](http://documentcloud.github.com/backbone/), a framework used widely throughout Readium. I am planning on explaining our decision to use Backbone in more depth in a future post, but for now the most important part to understand is its [Events module](http://documentcloud.github.com/backbone/#Events). Basically Backbone makes triggering and subscribing to custom event super simple. Here is a quick rundown of it in code:

```javascript
// create a vanilla Backbone model
var foo = new Backbone.Model();

// register a handler on foo
foo.on("banana", function() {
	// this code will execute when the banana event occurs
	alert('Wow, that was easy');
})

// fire the banana event on foo
foo.trigger("banana")
```

Instances of `Backbone.Model` also have a set of [observable](http://en.wikipedia.org/wiki/Observer_pattern) attributes, that is to say, they fire `change` events when they are changed _automagically_. The entire control flow of unpacking logic is driven by Backone events. Event firing happens both explicitly (ie `this.trigger("something")`) and implicitly by changing an observable attribute (ie `this.set("someprop", "newVal")`).

The next level in the hierarchy is `BookExtractorBase`. This is where all of the shared logic for parsing, managing and writing the imported EPUB to disk lives. I am going to come back to this in the next section.

At the bottom of the hierarchy are `ZipBookExtractor` and `UnpackedBookExtractor` each of these contains the logic for going through either a ZIP archive or unpacked EPUB and calling the necessary extraction / importing logic (located primarily in `BookExtractorBase`). Let’s consider `ZipBookExtractor`.

### ZipBookExtractor

Constructor logic is added to a `Backbone.Model` by overloading its `initialize()` method. The `Readium.ZipBookExtractor.Initialize` method does a few things:

1. It makes sure that the a URL has been passed in to the constructor, if not, it throws an exception.
2. If a URL has been passed in, it takes an MD5 hash of the string formed by concatenating a timestamp the the URL. This serves as a unique identifier, which readium uses to keep track of the file and also the name of the root filesystem directory Readium will write its contents to.
3. It captures the src filename of the users upload. This is captured only for the purpose of displaying it later in the UI, and is not needed for any technical reason.

Once a `ZipBookExtractor` instance has be created, the importing process is initiated by calling its `extract()` method. This method sets up the order in which the rest of the extraction process should execute by hooking up each method of the process to the event that will be fired upon completion of the previous step. Finally it calls the asynchronous constructor of `Readium.FileSystemApi`, which is our own very thin wrapper around the [HTML5 filesystem API](http://www.html5rocks.com/en/tutorials/file/filesystem/). 

Once the filesystem is opened, the process continues by calling the constructor of our zip library. This constructor is also asynchronous. When initialized, it executes callback with the `Zip` object as its `this` context. The `Zip` has not been decompressed at this stage, only the archive data structures have been parsed, so it does contain information about all of the archive entries. We decompress individual entries as needed (the code for this is in the `extractEntryByName` method).

### What to unpack first

After we perform some very rudimentary validations on the Zip object (mostly checking for the existence of a few essential items) we get into the brunt of unpacking the book. In general when unpacking an epub, what you want to get your hands on is the _OPF Package Document_ but first you need to find it. In order to find it you have to parse another xml file first: `META-INF/container.xml`. All epubs must have this file and it must be located at this absolute path in the archive. The `container.xml` specifies the absolute path of `package document` within the archive on an xml `<rootfile>` node. This is the only information that we take from this file.

### Parsing the package document

Once we are able to find the `package document` within the archive the next step is to parse it. The logic for this is contained in the [PackageDocumentBase](https://github.com/readium/readium/blob/master/scripts/models/packageDocument.js) class definition. This is separated from the unpacking code because we also parse package documents right before we display a publication. 

Whenever possible, we try to parse xml using a library called [Jath](https://github.com/dnewcome/jath). Jath allows declarative, template based parsing of xml into a structure of Javascript objects. Again I will differ the design justification for this to a later post.

At this stage when parsing the `package document` we are really only interested in the meta data that it contains. Essentially what we are looking for is everything we need to show the publication in the library view (title, author, cover image etc). Once we have this information, we add in a bit more of our own (most importantly where we are going to write the package document on disk) and then save it to the browser’s local storage for quick access. We then iterate through the remaining archive contents and extract them one by one and write them to the HTML filesystem.

### Monkey Patching

The last step of the unpacking process is something that I have termed __Monkeypatching the URLs__. There are a couple of reasons for this but the main motivation is that when we open an EPUB in the viewer, there are no guarantees that it will be loaded in a document with a URL pointing to where it actually exists on disk. The implication is that in order for us to ensure that URLs in individual spine items are handled correctly by the browser (for example consider the `src` attribute of an `<img>` tag) we need to convert them into fully qualified, absolute URLs, pointing to where we saved the file in the HTML5 filesystem. This logic is contained in [scripts/models/path_resolver.js](https://github.com/readium/readium/blob/master/scripts/models/path_resolver.js). This process is a little muddy right now and there are a [few bugs](https://github.com/readium/readium/issues/66) left to iron out (so if you are looking for some way to contribute :) this might be a good spot to jump in). 

### Time to open

Once the book is fully unpacked, monkey patched and written to disk, we fire off an event to let the library code know that we are done. In response we open the file in the viewer. How this happens, I will save for a future post.
