---
layout:     post
title:      "Post-Merge Blink Graphics Dependenciesy"
subtitle:   "Chromium Weekly 7"
date:       2015-11-9
author:     "geguangle"
header-img: "img/about-chromium.jpg"
tags:
    - Chromium
    - Architecture
    - 翻译、整理
---
> Here's a proposal for how the dependencies between Blink and Chromium (in particular, cc) that paint-dev cares about could evolve once the repositories merge.

> Let Source/platform/ include Chromium directories (especially cc/ and ui/gfx/), and gradually kill cc/blink/

Blink合并到Chromium的Repo之后，一些依赖可以简化。

合并之前Blink和Chromium隶属于不同的Repo，Blink不能直接依赖Chromium中的任何实现。

二者通过Blink API交互，主要是 public/platform 和 public/web。

Chromium通过实现public/platform为Blink提供功能，通过public/web使用Blink功能。

合并之前：

Chromium中的一些实现不单单是为了Blink，比如 [cc::Layer](https://code.google.com/p/chromium/codesearch#chromium/src/cc/blink/web_layer_impl.h&l=40&ct=xref_jump_to_def&cl=GROK&gsn=cc)就被 [blink::WebLayer](https://code.google.com/p/chromium/codesearch#chromium/src/cc/blink/web_layer_impl.h&l=28&ct=xref_jump_to_def&cl=GROK&gsn=blink)和ui::Layer使用。

这种情况下，cc专门有个目录cc/blink，放置相应实现
```
class WebLayerImpl : public blink::WebLayer, public cc::LayerClient {

...

 protected:

  scoped_refptr<cc::Layer> layer_;

...

}
```

下面图中，同一种颜色为一个Repo
![](https://docs.google.com/drawings/d/sm22do_iDdEOdwvbmSE7RvA/image?w=418&h=391&rev=2&ac=1)


这样如果Blink想使用cc::Layer的新接口，就需要在 blink::WebLayer和 cc_blink::WebLayerImpl增加和实现对应接口。

合并之后：
![](https://docs.google.com/drawings/d/s6RHsRzEQEp3wEv3bSG12ig/image?w=339&h=389&rev=2&ac=1)


Blink直接依赖cc，cc/blink中的实现也就可以慢慢去掉了。

类似的还存在gpu/blink，media/blink

