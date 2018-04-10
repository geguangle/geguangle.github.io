---
layout:     post
title:      "HTML5 by Default，Flash will gone--Q4 2016"
subtitle:   "Chromium Weekly 18"
date:       2016-6-20
author:     "geguangle"
header-img: "img/about-chromium.jpg"
tags:
    - Chromium
    - Architecture
    - 翻译、整理
---

## HTML5 by Default，Flash will gone--Q4 2016 
2016年底，HTML5将为默认设置，逐渐取代Flash。

> Later this year we plan to change how Chromium hints to websites about the presence of Flash Player, by changing the default response of Navigator.plugins and Navigator.mimeTypes.  If a site offers an HTML5 experience, this change will make that the primary experience.  



Chrome团队经过数据收集和验证，目前的计划如下:

> **Sep, 2015** - Plugin Power Saver (Peripheral)
> 
> **May, 2016** - Intersection Observer
> 
> **May, 2016** - (Experimental) WebAssembly support

> **Sep, 2016** - Plugin Power Saver (Tiny)

> **Q4 2016** - HTML5 by Default



详情参考[Intent to implement: HTML5 by Default](https://groups.google.com/a/chromium.org/forum/#!topic/chromium-dev/0wWoRRhTA_E%5B1-25%5D)、[Chrome, Flash, and the Future](https://docs.google.com/presentation/d/1iSTaGSUti9XNYXk6Bse578-M2FvSvS2v0Uxm6fAWmjw/edit?ts=5759e395#slide=id.p)

从webkit.org的博客中，发现苹果公司也有相关计划。

“在今年秋天推出的Safari 10浏览器中将默认不支持Flash插件，如果你之前的系统已经安装了Flash插件，那么在升级时将会被自动忽略。

> When Safari 10 ships this fall, by default, Safari will behave as though common legacy plug-ins on users’ Macs are not installed.

在同时提供Flash和HTML5的网站上，Safari将完全用HTML5来取代。

> On websites that offer both Flash and HTML5 implementations of content, Safari users will now always experience the modern HTML5 implementation, delivering improved performance and battery life. 


不管是默契，还是商量好的，面临2家巨头的联合行动，这次Flash恐怕要终结了。

详情参考[Next Steps for Legacy Plug-ins](https://webkit.org/blog/6589/next-steps-for-legacy-plug-ins/)


