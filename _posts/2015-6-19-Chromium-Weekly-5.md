---
layout:     post
title:      "Summary"
subtitle:   "Chromium Weekly 5"
date:       2015-6-19
author:     "geguangle"
header-img: "img/about-chromium.jpg"
tags:
    - Chromium
    - Architecture
    - 翻译、整理
---
### 一、PSA

1.[New Android APK: ChromePublic](https://groups.google.com/a/chromium.org/d/topic/chromium-dev/N6nnYxjFHDc/discussion)

> We’re excited tonight to have landed the majority of the remaining Chrome for Android code into the Chromium repo. This is over 100k lines of code, including the UI layer. You can build the new code using two new targets: chrome_public_apk and chrome_public_test_apk.

Chrome For Android的UI代码终于要放出来了。

[https://code.google.com/p/chromium/wiki/AndroidBuildInstructions](https://code.google.com/p/chromium/wiki/AndroidBuildInstructions)

2.Google I/O 2015 Codelabs

Google I/O 2015 Codelabs 是一套非常好的动手学习 Google 新技术的资源，内容包含 Android、Android Wear、Web、App Index、Unity(Cardboard, Project Tango)、Google Maps 等二十余个项目。选择你感兴趣的内容，按照教程一步一步执行就可以完成。例如 Web Animations 项目流程：
![](http://cl.ly/dwvp)

Codelab 教程：[https://io2015codelabs.appspot.com](https://io2015codelabs.appspot.com)
 
项目源码：[https://github.com/googlesamples/io2015-codelabs](https://github.com/googlesamples/io2015-codelabs)
 
### 二、Design

1.[Slimming Paint: stopping painting of content that is offscreen](https://docs.google.com/document/d/14N1aCYeB8NXs61oeeO0mlhPThzibeJibZ8hh6nSKIa4/)

为了对页面中离屏的动画GIF进行优化，进而引申到页面中其他离屏的内容（如图片），随着Slimming Paint对Blink的改造，特定的内容可以安全的跳过一些处理。
对网页另存为图片的影响？
 [discussion](https://groups.google.com/a/chromium.org/forum/#!msg/paint-dev/Bnr-YI38k3s/AYdsy94NS40J)
 [Issue](https://code.google.com/p/chromium/issues/detail?id=440466)
 
### 三、Tips

1.[Lossy prefs](https://groups.google.com/a/chromium.org/forum/#!topic/chromium-dev/yPFAhy-bGEQ)

增加了一个 [LOSSY_PREF](https://code.google.com/p/chromium/codesearch#chromium/src/base/prefs/pref_registry.h&l=43) flag，可以减少向prefs files写出的次数，某些不太重要的值可以使用此标识。
设置了此标识的值写出时机:

- a.[JsonPrefStore()](https://code.google.com/p/chromium/codesearch#chromium/src/base/prefs/json_pref_store.h&cl=GROK&l=43&ct=xref_jump_to_def&gsn=JsonPrefStore)析构的时候
- b.不带此标识的值写出时，会将其一起写出。
 
2.[How to tell if a WebElement is visible?](https://groups.google.com/a/chromium.org/forum/#!topic/chromium-dev/2KmWAWUD4AM)

看起来，如何判断一个元素对用户是否可见是困难的。我们在改进密码保存功能也有这个困惑，一些站点动态的设置用户名和密码框的显示状态，或者在页面中插入一些不可见的用户名和密码框。这些噪音对识别正确的用户名和密码框影响非常大。现在是增加检测显示相关的属性，增加识别的准确性，效果还行，锵锵够用。
 
3.同源策略影响download属性
```
<a class="video-link" href="http://download-url" download="Painting in Chromium" 
target="_blank">WEBM</a>
```
形如这样的元素，下载文件名默认为Painting in Chromium，但是如果href地址和当前打开地址不是同源，则忽略download属性中的值。

![](http://cl.ly/dxEX)

4.Cr-Blink bug status as of 2015-06-15
![](http://cl.ly/image/2831403U1y2Q/Image%202015-11-30%20at%201.41.06%20%E4%B8%8B%E5%8D%88.png)

5.Contribution for chromium

希望志愿者来解决的
[https://code.google.com/p/chromium/issues/list?q=label:HelpWanted](https://code.google.com/p/chromium/issues/list?q=label:HelpWanted)

解决BUG先从这样的入手
[https://code.google.com/p/chromium/issues/list?q=label:Hotlist-GoodFirstBug](https://code.google.com/p/chromium/issues/list?q=label:Hotlist-GoodFirstBug)


