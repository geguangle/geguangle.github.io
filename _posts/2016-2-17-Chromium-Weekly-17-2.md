---
layout:     post
title:      "Smooth Scrolling in Chromium"
subtitle:   "Chromium Weekly 17"
date:       2016-6-20
author:     "geguangle"
header-img: "img/about-chromium.jpg"
tags:
    - Chromium
    - Architecture
    - 翻译、整理
---

[Smooth Scrolling in Chromium](https://docs.google.com/document/d/1JQ6jLy-r7vw_I9s3rtWAIK137fMpF_OFVtfVsgOaoTA/)

----------

实验性功能，通过enable-smooth-scrolling控制。
目的就是为了提高用户体验，在鼠标滚轮、键盘方向键、PageUp、PageDown等情景下，将滚动分成多步进行，给人感觉上更加平滑。
影响：

1.鼠标滚轮滚动一次时，页面中的JavaScript会监听到很多滚动事件，并且适时更新scrollLeft / scrollTop。

2.滚轮事件和完成滚屏之间会有一些延迟。

