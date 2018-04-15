---
layout:     post
title:      "Summary"
subtitle:   "Chromium Weekly 6"
date:       2015-7-16
author:     "geguangle"
header-img: "img/about-chromium.jpg"
tags:
    - Chromium
    - Architecture
    - 翻译、整理
---

### 一、PSA
1.[Web-facing PSA: stop sending mouse position updates while scrolling](https://groups.google.com/a/chromium.org/forum/#!topic/blink-dev/KIoVljZw5fc)

> While scrolling via trackpad or mouse wheel, we currently send mouse position updates every 100ms. On pages with heavy mouse handlers or :hover styles, this can cause significant amounts of scroll jank.

> Sending a mouse position update includes updating :hover styles, and dispatching mousemove, mouseover, mouseenter, mouseleave, and mouseout events.

页面滚动中，停止发送鼠标位置的信息；页面停止滚动100ms后，再允许发送鼠标位置的信息。这样滚动过程更加顺滑。

2.[Generate the icu data binaries at compile time instead of checking in binaries](https://codereview.chromium.org/1000163003/)

icu data将改为编译时生成，修改中，还未提交。

### 二、Design

1.[Slimming Paint: stopping painting of content that is offscreen](https://docs.google.com/document/d/14N1aCYeB8NXs61oeeO0mlhPThzibeJibZ8hh6nSKIa4/)

实现的关键点，在Slimming Paint开启的情况下，如果是不可见的动画，则跳过此次Invalidation，并启动一个定时器，定时看一下，可见性是否发生了变化。
 

2.[Intent to Implement: Token Binding](https://docs.google.com/document/d/1Ta3GlT_LrqAOLV217Kutn3B2trvifStxB0CThQ_kk78/)

动机：
现在许多网络服务都使用cookies, OAuth tokens等来对客户端进行认证。但是这中方式有可能被“中间人”攻击。鉴于此，Token Binding 就是用来解决这一问题的。
 
协议支持：
> a)HTTP/1.1 over TLS, SPDY
> 
> b)HTTP/2
> 
> c)暂时未考虑QUIC
 
实现：

涉及TLS层和应用层的改变，TLS提供tls_unique和控制Token Binding的启用与关闭；应用层用tls_unique生成TokenBindingMessage，在HTTP headers添加Token-Binding字段，应该是形如:
Token-Binding：TokenBindingMessage

### 三、Tips

1.[What is the preferable string format for code used on multiple platforms?](https://groups.google.com/a/chromium.org/forum/#!topic/chromium-dev/U4lqhf-1SBM)

> std::string - single byte, might be UTF-8, might be not。

使用场景：内容全部为英文、函数调用过程中

> string16 - double byte - UTF-16。

使用场景：内容可能含有非英文、序列化时、网络传输时

> std::wstring - double byte on Windows (UTF-16), quad byte on Linux(UTF-32), ? on Mac。

使用场景：不建议使用。

2.[static initializers](https://groups.google.com/a/chromium.org/forum/?fromgroups#!topic/chromium-dev/72lluqLr46Y)

以前的认识是，全局非POD静态变量，由于初始化顺序未定义，这种情况务必要绕道而行，避免依赖这种设计。
 
这里讨论的是，全局非POD静态变量太多，导致加载时间变长。
帖子中的介绍，做完处理后，相关的加载量由26MB降低到了19.5MB，可谓效果显著。
 
总结，全局非POD静态变量：初始化顺序未定义、影响模块加载时间。
 
纵观代码库，误用场景如下

a) Including <iostream>
```
This file has this in it:
// For construction of filebuffers for cout, cin, cerr, clog et. al.
static ios_base::Init __ioinit;
```
这个Code Sytle里也提到了，头文件不要包含<iostream>
 
b) Statically declaring a SkColor variable with the result of SkColorSetRGB() or SkColorSetARGB()

这个是把SkColorSetRGB用宏包了一下就可以了，`但是没看明白为什么`。他们是查看汇编来确认生效的，没提到原理。

c) Using std::string or std::wstring when we should be using char or wchar_t for string constants.

d) Using a static class instance of another kind as a variable, when it should instead be contained within a static function with a local static variable

e) Using a static scaler but setting its value to the result of a function call that can't be computed at compile time, such as string16.

下面是相关的代码提交

[LoadLibrary for chrome.dll takes 500+ ms on average in cold mode](https://code.google.com/p/chromium/issues/detail?id=48097)

> The startup times for chrome in cold are around 1.5 to 2 seconds. LoadLibrary for chrome.dll takes around 30% of the total time.

> LoadLibrary after resolving import dependencies invokes the entry point of chrome.dll where it initializes a large number of global objects. These objects are spread across the code base and a majority of these objects have non trivial constructors. List below:
> 1. Skia.
> 2. libjingle.
> 3. Protocol buffer.
> 4. Sync.
> 5. Chrome browser sources.
> 6. Some objects in the Chrome plugin and renderer code.

> It appears that a large chunk of the LoadLibrary time is actually spent paging in the relevant code pages because of poor code organization.

> We should look into reducing the number of static initializers and also organize the code better to achieve better performance.






