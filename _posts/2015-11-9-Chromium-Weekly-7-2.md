---
layout:     post
title:      "PSA: Change to the //components policy"
subtitle:   "Chromium Weekly 7"
date:       2015-11-9
author:     "geguangle"
header-img: "img/about-chromium.jpg"
tags:
    - Chromium
    - Architecture
    - 翻译、整理
---
components中的组件是从content抽离出来的，故依赖关系是`chrome->components->content`。

html_viewer是基于mojo的一个Blink Embedder，这样实现Blink特性的组件也将放入components目录，和以前相比，称为new style。
 
那么content是可以依赖这样的组件的，但是这样components就显得很乱了， Layer Policy会把人搞晕了。

讨论成了一锅粥，没有看到定论，而且`https://www.chromium.org/developers/how-tos/html_viewer`还被拿掉了，页面不存在了。

不过倒是弄明白了另一件事， html_viewer是干什么的？ 她基于mojo，是个mojo应用，把自己注册可以处理text/html等类型，这样了mojo_shell打开网页时，就会调起html_viewer了。

后记：

是否可以这样呢增加blink_components:`chrome->components->content->blink_components`





