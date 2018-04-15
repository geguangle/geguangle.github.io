---
layout:     post
title:      "SIMD.js & base::Feature API"
subtitle:   "Chromium Weekly 9"
date:       2015-11-27
author:     "geguangle"
header-img: "img/about-chromium.jpg"
tags:
    - Chromium
    - Architecture
    - 翻译、整理
---

### 一、[PSA] Wiki documentation is being migrated off of code.google.com

> We will replace the wiki with a smaller, more focused set of documentation written in Markdown that lives within the source.

借着从code.google.com的迁移工作，一些新的文档和对老文档的更新，都将使用Markdown来写。代码目录中搜索后缀.md，已经有不少文档了。大家可以了解一下了，方便对文档的阅读。###

### 二、SIMD.js has now been implemented in v8，behind flag --harmony-simd
 SIMD 的全称是 `Single Instruction Multiple Data`.

> SIMD.js exposes hardware SIMD instructions to Javascript applications in a platform-independent way. SIMD.js can speed up applications that process lots of data that is naturally organized as 2, 3, or 4 element vectors. SIMD is also useful when compiling native code to Javascript for static compilation by the Javascript engine.###

参考[https://www.chromium.org/developers/how-tos/run-chromium-with-flags](https://www.chromium.org/developers/how-tos/run-chromium-with-flags) 可以这样打开这个flag。

```
chrome.exe --js-flags="--harmony-simd"
```

### 三、Clean up & document macros in Blink

> Allow only wtf/ to use macros defined in src/base/. In this case, wtf/ defines a thin wrapper for the macro in src/base/ (e.g., WTF_DISALLOW_COPY_AND_ASSIGN). The rest of Blink should use the think wrapper defined in wtf/.

> Currently including base files is not allowed under third_party by default, we could create almost empty DEPS under wtf/ and start adding files for whitelisting.

随着Blink和Chromium的整合，从代码的风格到重用都会越来越统一和一致。wtf作为一个包装层，先来粘合二者，而不是整个Blink全压上来。Blink依赖wtf，wtf/依赖src/base，最后再做统一。


### 四、Announcing base::Feature API

新鲜出炉的，用于增加特性开关。声明特性时，可以设置默认状态：
```
const base::Feature kMyFeature {
  "MyFeature", base::FEATURE_ENABLED_BY_DEFAULT`
};
```
检测特性时：
```
base::FeatureList::IsEnabled(kMyFeature)
```
可以通过命令行修改
```
--enable-features=MyFeature
--disable-features=MyFeature
```
这就比原来方便多了，不然你就得增加2个开关
```
--enable-myfeature
--disable-myfeature
```
以后尽量用这个吧，她的位置 `base/feature_list.h`

### 五、Chrome Dev Summit 2015#
大部分都是前端开发相关的，大家可以了解一下。[https://www.youtube.com/playlist?list=PLNYkxOF6rcICcHeQY02XLvoGL34rZFWZn](https://www.youtube.com/playlist?list=PLNYkxOF6rcICcHeQY02XLvoGL34rZFWZn)



