---
layout:     post
title:      "Chromium for iOS 终于全部开源了"
subtitle:   "Chromium.app for iOS is now available"
date:       2016-12-28
author:     "geguangle"
header-img: "img/about-chromium.jpg"
tags:
    - Chromium
    - Architecture
    - 翻译、整理
---

至此，```Chromium```在六大平台都是全部开源了。

这一天终于来了！


下面是官方原文：

```
Hi all,

We are happy to announce that the majority of the Chrome for iOS code
has moved into the Chromium repo.  We moved over 200k lines of code,
including the UI layer.  You can try it in an iOS checkout by building
the “chrome” target and running the resulting “Chromium.app”.  Support
for this target has automatically been added to our gn-generated Xcode
project.

We have also expanded our use of the EarlGrey test framework to add
several new suites of tests for Chromium.app.  These tests currently
run on the EarlGreyiOS bot on chromium.fyi, but we plan to move them
over to the main waterfall once they’re running smoothly.

We’d like to thank everyone who helped with this effort, including
everyone who helped componentize code over the past few years.  We
would not be in the public repo today without your contributions.
```

作为```PC```端开发者，每次学习移动端浏览器技术时，都需要熟悉新的技术栈，从概念到技术实现来回切换，很是难受。

现在好了，让我们快看看开源到什么程度了吧。


```
$ mkdir chromium && cd chromium

$ fetch ios

$ cd src

$ python ios/build/tools/setup-gn.py
```
顺利拉取到最新代码，最后一步会在out目录生成工程文件。
![](http://p1.bpimg.com/567571/72f45efdd7161b9b.jpg)

相对```PC```编译速度还是很快的。
![](http://p1.bpimg.com/567571/0cd8b79f689d9411.jpg)

非常顺利，让我们跑来~~
![](http://p1.bqimg.com/567571/82b28b7090439fd8.jpg)

![](http://p1.bqimg.com/567571/9ed912fb3d0238a9.jpg)

![](http://p1.bqimg.com/567571/3f889b002dbf18fd.jpg)

正如官方所说，除了```Google```特有的相关服务，全部开源了，```iOS```又有的学习和借鉴了。

