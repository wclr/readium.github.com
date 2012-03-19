---
layout: post
title: "Readium Adds MathML Support by Integrating MathJax"
date: 2012-03-18 17:11
comments: true
author: Peter Krautzberger
categories: 
---

Readium now renders MathML by integrating MathJax. [MathJax](http://www.mathjax.org) is an open source, cross-browser JavaScript library sponsored by the American Mathematical Society, Design Science, and the Society for Industrial and Applied Mathematics as well as [several partners](http://www.mathjax.org/sponsors/).

Work on native support of MathML in WebKit [has begun](https://trac.webkit.org/wiki/MathML) but is a long way from being ready to use. With MathJax, Readium will be able to render MathML, an important step towards Readium's full EPUB 3 support. Additionally, MathJax offers zoom for accessibility and cut-and-paste of mathematical content.

The necessary code has been pulled into the main branch and Readium now displays math beautifully. You can find a sample at [here](https://github.com/dpvc/readium/blob/mathjax/examples/sample-tex-mml.epub).

Thanks to Davide Cervone, MathJax's Lead Developer, for his help in making MathJax work in Readium.