---
layout:     post
title:      "Closure Compiling Chrome Code"
subtitle:   "Chromium Weekly 12"
date:       2015-12-17
author:     "geguangle"
header-img: "img/about-chromium.jpg"
tags:
    - Chromium
    - Architecture
    - 翻译、整理
---


```
What is the Closure Compiler?

The Closure Compiler is a tool for making JavaScript download and run faster. It is a true compiler for JavaScript.  
Instead of compiling from a source language to machine code, it compiles from JavaScript to better JavaScript. It  
parses your JavaScript, analyzes it, removes dead code and rewrites and minimizes what's left. It also checks   
syntax, variable references, and types, and warns about common JavaScript pitfalls.  
```

Closure Compiler可以发现一些错误，优化代码，压缩代码。不过Closure Compiler是通用的，要想让她认识chrome.send等Chrome特有的代码，需要做一些特化。
代码库中还不能使用此功能，[谷歌提供了在线服务](https://developers.google.com/closure/compiler/?hl=en)

背景：Chrome中大量WebUI，使用了HTML/CSS/JS

- the New Tab page
- settings
- history
- downloads
- bookmarks
- extensions
- about/help page
- the login screen/file manager on ChromeOS
- print preview

现状：

- 非类型安全的，运行时报错。
- 自动化测试，不能覆盖所有路径。
- 开发人员，不能愉快的重构代码。
- JS代码未做优化和压缩。

目标：减少回退、提高开发效率、减少安装包体积。


