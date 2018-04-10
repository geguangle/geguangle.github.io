---
layout:     post
title:      "GN is coming@M53"
subtitle:   "Chromium Weekly 18"
date:       2016-6-20
author:     "geguangle"
header-img: "img/about-chromium.jpg"
tags:
    - Chromium
    - Architecture
    - 翻译、整理
---

## GN is coming@M53
随着GN成熟度越来越高，**M53**之后，将全面进入GN时代，下面是目前的计划：


|Platform|Owner| Ship with GN | Ready for developers | All bots flipped/GYP turned off |
| -- | -- | -- | -- | -- |
| Linux     | dpranke | Done | Done       |May end (almost done here!) |
| Android   | agrieve、jbudorick | Done | Done       | Done                       |
| Windows   | brucedawson | M53  | Done       |          --                  |
| ChromeOS  | stevenjb | M53  | 3:5        |           --                 |
| Mac       | rsesek | M53  | Done       |             --               |
| iOS       | sdefresne | M53  | End of may | -- |

咱们也要着手学习一下GN了，下面是官方的一个入门展示

[Using GN build](https://docs.google.com/presentation/d/15Zwb53JcncHfEwHpnG_PoIbbzQ3GQi_cpujYwbpcbZo/edit#slide=id.g119d702868_0_12)

或者从源码中的说明入手也可以。

> tools/gn/README.md
> 
> tools/gn/docs/

M50已经可以使用GN了，试用了一把，截图如下：

![](/img/18/gn.png)

![](/img/18/gn_out.png)

![](/img/18/gn_solutions.png)
总的来说
1. GN比GPY速度要快很多
2. 非常方便自定义输出目录
3. 生成的工程文件全都放在输出目录中，不再和代码混在一起

