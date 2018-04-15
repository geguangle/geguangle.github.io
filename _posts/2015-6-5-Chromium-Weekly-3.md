---
layout:     post
title:      "Summary"
subtitle:   "Chromium Weekly 3"
date:       2015-6-5
author:     "geguangle"
header-img: "img/about-chromium.jpg"
tags:
    - Chromium
    - Architecture
    - 翻译、整理
---

### 一、Change
1.Binding team 2015

`Goals:`
* more conformant with the spec (Capabilities)
* easier to use (Internal productivity)
* faster (Performance)

`Content:`
* More conformant with the spec
* Support more IDL features
* Make existing Blink IDLs more conformant

2.Blink componentization

Define good dependency rules among the components
![](http://cl.ly/dx3Y)
Fig.1.1 The current architecture

Make it much easier and faster to implement chrome/content/ ⇔ Blink interactions

Fig.1.2 The proposed architecture
![](http://cl.ly/dwfY)

Reduce the binding overhead  

* Reduce # of CPU instructions in the binding code
* Remove TryCatch blocks
* Remove redundant branch conditions

2.Blink componentization after the Chromium-Blink repository merge
> This document proposes a way to re-architect Blink directories and their dependencies after the repository merge and the core-modules componentization. The goal of the re-architecture is to remove unnecessary layering abstractions between Chromium and Blink and make the interaction *simpler* and *faster*.

![](http://cl.ly/dwya)
Fig.2.1 The current architecture

![](http://cl.ly/dwzT)
Fig.2.2 The proposed architecture

`public/web/ contains WebXXX classes exposed from Blink to content/`.

`public/delegate/ contains WebXXX classes exposed from content/ to Blink`.

这两篇说的都是Blink的变化，第一篇更侧重一些Blink内部自身的演变，第二篇更加关注Blink和Chromium的共同演进。Blink的一些变化不可避免的会影响到她的使用者，比如两篇都涉及的Blink的组件化。Webkit大体是清晰的，但是着眼于实现，依赖关系错综复杂，由Webkit而来的Blink可以甩掉一些历史包袱，发力于更好的为Chromium提供服务，和利用Chromium中更多的代码库。

有意思的是，关于Blink组件化采用的方案，选择了不同的答案Fig1.2和Fig2.2。这2个答案haraken@chromium.org应该都知晓的。

### 二、Design
1.Scheduling JS Timer Execution

`Goal`
> Reduce Jank, input latency and queueing durations by allowing the scheduler to defer javascript timer execution.

> Input tasks and (sometimes) compositor tasks are on the critical path for smoothness.
> Javascript supports several APIs (setTimeout and setInterval) collectively known as DOMTimers.   While these can block the main thread for an arbitrary length of time, this document is focused on fixing problems cause by tasks running at unfortunate times.  

对用户输入的响应速度，事关感受上的流畅性。这篇主要就是把Input tasks和Compositor tasks优先级提高，JS Timer放入单独的队列，并对其优先级降低。尽量放在Idle时间来执行它。这个优化主要是针对移动端的。
![](http://cl.ly/image/3G3Q2z283X2B/Image%202015-11-30%20at%201.58.55%20%E4%B8%8B%E5%8D%88.png)

### 三、Tips
1.Defining a new attribute for Script tags in chrome
> 1)src/third_party/WebKit/Source/core/html/HTMLAttributeNames.in
==> added newAttr;

> 2)src/third_party/WebKit/Source/core/html/HTMLScriptElement.idl
==> added a new field in the HTMLScriptElement interface as below
attribute DOMString new;

2.Tracking slimming paint performance
> I'm going to continue manually running cluster telemetry runs on the top 10k pages every morning.

3.Scheduling JS Timer Execution
> The chromium side of the Workers needs to live in content/child which isn’t allowed to include files from content/renderer (the reverse is allowed).

