---
layout:     post
title:      "Implementing a layered web platform"
subtitle:   "Chromium Weekly 12"
date:       2015-12-17
author:     "geguangle"
header-img: "img/about-chromium.jpg"
tags:
    - Chromium
    - Architecture
    - 翻译、整理
---

`目标：`

1.High-Level功能可以通过Primitives实现。

2.Low-Level功能准确表达High-Level意图。

3.保持良好的分层，Low-Level对High-Level无感知。

这样可以让我们基于低层的原语，更快的完成高层功能，而不必深入平台级的细节。开始时，没有原语可用，速度可能会慢，但是随着原语的增多，长期来看，速度是越来越快的。

`建议做法：`

1.基于web-exposed IDL自动生成gen/webapi/，这里的API同时暴露给C++和JavaScript。

2.在webmodules目录中，基于原语实现高层API，这个目录只能从上面的webapi和其它webmodules引入头文件。

`以snap-points为例：`

- Implement CompositorWorker in core
- Implement scroll customization in core
- Implement custom layout in core
- Implement custom CSS properties in core
- Expose the above to C++ via gen/webapi
- Implement snappoints on top of the above in webmodules

这是向着目标迈出的一小步，最终我们会将所有的C++代码都使用JavaScript来实现。这样就可以提供一个标准的库，可以运行在所有的浏览器平台。到时库里的动能可以按需开启，让运行成本更低。

既然分层了，层与层之间解耦了，每一层都可以做到标准化，每一次可被替换了。问题是，标准化从来都是个很难的事情，何况是大佬之间达成一致。虽然有点理想化，但是万一实现了呢，到时Web开发更加容易了。

