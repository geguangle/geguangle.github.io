---
layout:     post
title:      "PSA: Windows now builds against the Windows 10 SDK"
subtitle:   "Chromium Weekly 7"
date:       2015-11-9
author:     "geguangle"
header-img: "img/about-chromium.jpg"
tags:
    - Chromium
    - Architecture
    - 翻译、整理
---

When building on Windows, Chrome now builds against the Windows 10 SDK.
If you use the automatic toolchain (most devs at Google and all bots) then you don't need to do anything.
Otherwise, you'll need to make sure you have installed the SDK in its default install location `"C:\Program Files (x86)\Windows Kits\10"`.
This doesn't change anything about what platforms Chrome runs on.

但是发现有点问题，暂时回退了，应该用不了多久会杀回来。





