---
layout:     post
title:      "消失的Mandoline"
subtitle:   "Chromium Weekly 19"
date:       2016-12-10
author:     "geguangle"
header-img: "img/about-chromium.jpg"
tags:
    - Chromium
    - Architecture
    - 翻译、整理
---

### 简读版

何为```Mojo```

>Mojo is a platform for sandboxed services communicating over IPC. It is based on experience and infrastructure we built in Chromium to support our multi-process architecture.

```Mojo``` 主要提供了 ```Sandbox、IPC、Multi-process``` 底层架构实现，可以接手 ```content``` 的部分责任。

随着 ```Mojo``` 的展开，让 ```Chrome``` 团队产生了一些新的设想，说干就干，快速抽调几个人，成立 ```Mandoline``` 项目，基于 ```Mojo``` 专门用来验证那些想法是否可行。经过一段时间的摸索， ```Chrome``` 团队找到了一个新的方向 -- ```Service-Oriented Model```。

未来的几年里，```Chromium``` 的架构将会朝着这个方向演进。


----------

我们知道，```Chromium``` 的架构是一直不断地演进，现在她又迎来了一次大的变化。

让我们先简单回顾一下前几次变化

> Content Module
> 
> Components
> 
> Layered Components

我们知道，```Chromium``` 项目最初的支持平台只有 ```Windows```，设计相对简单，```All In One```，大部分代码都放在了 ```chrome``` 这一层。
![](http://i.imgur.com/Xqr4Vx6.png)

随着项目的演进，```chrome``` 中的代码迅速膨胀，其中架构和功能方面的代码交织在一起，开发、维护都非常难受了。

2011-1-13，通过专项 ```Project Carnitas```，将多进程，沙箱机制等稳定的架构抽离出来，组成新的 ```content``` 层，对外提供 ```Content API```，```Chrome``` 相关的功能留在 ```chrome``` 层，通过 ```Content API``` 进行交互。
![](http://i.imgur.com/Yi6R0hv.jpg)

随着支持的平台越来越多，囿于平台间的差异，```Chrome``` 中同一个功能，在不同平台的实现都会有些差异，这样下去，会涌现大量的 ```#ifdef```。 怎么办？将各个平台都需要的功能从 ```chrome``` 层中抽离出来，以 ```component```形式组织，平台间共享的实现放在 ```component``` 层，平台相关的实现放在 ```chrome``` 层中。

![](http://i.imgur.com/FR34DTm.jpg)

以 ```Autofill``` 的组件化为例子，前后对比如下：

![](http://i.imgur.com/pD2ovZd.jpg)

![](http://i.imgur.com/iWux7w5.jpg)

由于 ```iOS``` 平台上， ```Apple```的种种限制： 自家提供的渲染引擎和JavaScript引擎，必须为单进程， 导致 ```chromium``` 就不能依赖 ```content``` 层了。```Layered Components``` 就是将 ```component``` 中依赖 ```content``` 的代码继续剥离出来。可以和 ```iOS``` 平台共享的代码放入 ```core``` 文件夹，```iOS``` 特有的代码放入 ```ios```文件夹，依赖 ```content``` 的代码放入```content```文件夹。
通过```DEPS```机制来维护这种限制。

一路走来，一路演化，```Chromium``` 项目终于顺利支持了六个平台：```Windows、Mac、Linux、Chrome OS、Android、iOS```。

回首望望代码，复杂、臃肿、冗余充斥了各个角落，再出发，```Mandoline``` 实现项目应运而生。

----------

#### FYI - src/mandoline @ 2015-5-15 
> A few of us have begun investigating a replacement for src/content using Mojo and Mojo Shell. Our equivalent of content shell is in the new src/mandoline directory.
>
> The main effect of this on Chromium source are:
>
> - an additional Blink embedder in src/components/html_viewer (actually this has been there already for 12+ months)
> - componentization of features from src/content as we improve the Mojo blink embedding and want to share code with content.
>
> As we develop any useful docs/data/etc we will link them here:
> http://dev.chromium.org/developers/mandoline
>
> Thanks,
> -Ben

```Mandoline``` 项目发布于2015-5-15，一个用于验证架构改造可行性的项目。

目的是为了基于 ```Mojo```之上，重构 ```content``` 层，在架构上寻找更好的演进方向。

简单来说就是，```component``` 层直接和 ```blink``` 交互，来实现功能，而现在只能通过 ```content``` 来和 ```blink``` 交互。

以 ```src/components/html_viewer``` 为例，她通过 ```Mojo``` 直接利用 ```Blink``` 实现网页浏览、PDF查看。从 ```Mojo``` 角度来看就是，```Blink``` 以 ```Mojo``` 的形式提供服务， ```html_viewer``` 和 ```content``` 作为 ```Mojo shell``` 使用服务。

以前：

> Blink + Content + Component = Chrome

现在：

> Blink + Component = html_viewer

通过实践和验证，最后的结果非常不错，找到了 ```Chrome Service Model``` 的方向,这个的前提是实现 Mojo的迁移。

功成身退，```Mandoline``` 不再占用任何字节，消失的无影无踪。

```
Bye bye Mandoline

This nukes mandoline, html_viewer and web-view.

Review URL: https://codereview.chromium.org/1677293002
```

通过实现一个简单的 ```Shell``` 来验证可行性是一回事，用来实现一个真正的浏览器将会是另一回事。

```Chrome``` 吹响了前进的号角，这次重构的规模之大，将会使挑战不断出现。

这条路上，你我都将战于其中，就看相逢于何时了。

#### [Announcing the Content Modularization Project](https://groups.google.com/a/chromium.org/forum/#!topic/chromium-dev/_tWwQxxqDpA)

转眼5年过去，```Chrome``` 的功能大幅增加，```content``` 层负责的内容非常多，也变的非常复杂了，几乎所有的功能都会涉及到她，这里已经成为了目前的瓶颈。

我们能否减少这里的复杂性，简化目前的设计，让大家更加舒服的开发呢？

2015年，通过 ```Mandoline``` 项目进行了相关实验，基于面向服务的架构，做出了一个浏览器外壳，验证了这种方式是可行的。

这次将从 ```content``` 层开始，逐渐地将 ```chrome、content``` 的实现以 ```Mojo``` 的机制进行模块化。

当前的首要任务就是尽快将 ```Chrome``` 迁移到 ```Mojo``` 之上，新功能将会完全基于 ```Mojo```，老功能逐步开始迁移。
我们会从基础服务开始：

> UI(rendering & input)
>
> Network
>
> FileSystem
>
> Preference
>
> Local Storage
>
> ......

这个改动比之前的都要大，可以说是脱胎换骨，预计需要几年的时间才能完成，最终 ```content``` 层也许会消失。

现在新的设计方案基本都会涉及到 ```Mojo```，如果是相对独立的功能，基本都是 ```Chrome Service Model + Mojo``` 的风格。
当前的架构结构：
![](http://i.imgur.com/lVr13wi.png)

```Chrome Service Model``` 后的架构：
![](http://i.imgur.com/8RqeXOK.png)

目前新的设计文档都绕不过 ```Mojo``` 和 ```Chrome Service Model```， 本篇先开个头，做个铺垫，更多的是概念和背景。
后续用实例来介绍  ```Mojo``` 和 ```Chrome Service Model``` 在代码中的实现。

预计会从下面中选择内容来进行分享：

> Mojo 101 -- 需要请教@舒畅来帮忙
>
> Mandoline 设计及实现
>
> Network Service in Chrome
>
> LocalStorage design using Chrome Services
>
> Video Capture Mojo Component


有了这些背景知识，咱们就可以继续讲讲 ```Headless、Onion Soup、Mojo in Blink```等内容。

参考文献

> [Life of a
Browser Component](https://a77db9aa-a-7b23c8ea-s-sites.googlegroups.com/a/chromium.org/dev/developers/design-documents/cookbook/Life%20of%20a%20Browser%20Component.pdf?attachauth=ANoY7cotUtC8u2lRinQ2yBKfVGhjelmfcd0tGDjqyUH4uqNZRCscRZtI6OcB-2bSTRMbZxqDChU9m8sUWMRe9-xchSwYhkqv7ciVuAZP9KOzLU7SXHdbHPALNbpulaeuCjs1M8O40gYza1nX6z72ea_BiVbj0xcyTlr0LJU6fiEKYkBpeR3nAbRZq-qh4PO6AEGz7TzpbCV6hUfzz1b9GEvYSlSMm_UqzAVMGbjBCT2CxA7TBGr5-Y6-OyJ5hsFXPbimd2NJOtiNQ9QRpEil50hXvFUZmuYPTQ%3D%3D&attredirects=1)
>
> [Chrome Service Model](https://docs.google.com/document/d/15I7sQyQo6zsqXVNAlVd520tdGaS8FCicZHrN0yRu-oU/edit#heading=h.p37l9e7o0io5)
>
> [LocalStorage using Chrome Service Model](https://docs.google.com/document/d/18wpuaitdvki6mO4aWkjvORgryUHM-mvDIqppDXhbSTk/edit#heading=h.qf9i7c52uy3r)
>
> [Network Service in Chrome](https://docs.google.com/document/d/1wAHLw9h7gGuqJNCgG1mP1BmLtCGfZ2pys-PdZQ1vg7M/edit#heading=h.qf9i7c52uy3r)
>
> [Video Capture Mojo Component](https://docs.google.com/document/d/1Qw7rw1AJy0QHXjha36jZNiEuxsxWslJ_X-zpOhijvI8/edit#heading=h.2ki8zk71f7w2)
>
> [Mojo](https://drive.google.com/drive/folders/0B-WCZkfLIXQTajI4WWZhdGhjNnM)
>
> [Mandoline Project Summary](https://docs.google.com/document/d/1AjTsDoY6ugaykfqGLyOHYfp67hMp0tMjDbZcJ5EH9fw/edit?usp=sharing)
> 
> [Project Carnitas - Code Cleanup](https://groups.google.com/a/chromium.org/d/msg/chromium-dev/X4VTnSWRasE/J8E-VTqXyvgJ)
>
> []()


----------

### News Express

#### 1. [LUCI Project](https://groups.google.com/a/chromium.org/forum/#!topic/chromium-dev/YMJaQ0oXiVE)

> Layered "Universal" Continuous Integration - A collection of scalable open-source build infrastructure services.
>
> [LogDog](https://luci-logdog.appspot.com/) and [Milo](https://luci-milo.appspot.com/) are components of the LUCI project, an effort to provide more reliable and scalable build and test infrastructure for Chromium by augmenting and eventually replacing Buildbot. For more information see [github.com/luci](http://github.com/luci) or reach out to [infra-dev@chromium.org](infra-dev@chromium.org).

```LUCI``` 项目是```Chrome```团队在持续集成方面继续优化和改进，以期提供更加可靠的构建和测试设施。最终会取代目前的 ```Buildbot```。


#### 2. [PSA: Deprecate Blink-in-JS](https://groups.google.com/a/chromium.org/forum/#!topic/blink-dev/f430AkW9MTM)
2年前发起的 ```Blink-in-JS``` 项目已经被停掉，这个设计存在性能、安全方面的问题。

目前已经成立 ```Onion Soup``` 项目， 推动 ```Blink``` 向着层次化、服务化方向发展。这个新的设计方案，比 ```Blink-in-JS``` 更加优秀，后面会把 ```Blink-in-JS``` 已经实现的功能，重新用 ```C++``` 来实现。

#### 3. [IPC Message is deprecated](https://groups.google.com/a/chromium.org/forum/#!msg/chromium-dev/3zK8HWY10eo/tw1cIumKBQAJ)
>TL;DR: IPC::Message is deprecated. All new IPC in Chrome should use Mojo.

```Mojo``` 经过长时间的应用，已经达到非常好的状态。特别是在使用过程中，根据反馈还一直不断地改进，现在已经打磨的非常好了。咱们继续向前走，肯定是要切到这上面来的。官方说的优点那是相当多，目前我也不懂，等后面咱们结合代码、案例再展开。

