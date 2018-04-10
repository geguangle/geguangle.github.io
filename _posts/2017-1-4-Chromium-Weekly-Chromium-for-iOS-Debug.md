---
layout:     post
title:      "Chromium for iOS 如何调试"
subtitle:   "How To Debug Chromium for iOS"
date:       2017-1-4
author:     "geguangle"
header-img: "img/about-chromium.jpg"
tags:
    - Chromium
    - Debug
    - iOS
---

上次说到```Chromium for iOS```全部开源了，这么好的资源，当然要扑进去学习了。

代码编译搞定后，满心打开```XCode```准备好好调试一番的，可惜遇到好多问题，无奈无奈。
![](/img/chromium_ios_debug/out.jpg)

首先打开```out```目录下的工程文件就报权限问题，手动改相关目录为可读写，结果还是不行。
![](/img/chromium_ios_debug/lock.jpg)

最后通过命令行启动```XCode```绕过这个坑。

```sudo /Applications/Xcode.app/Contents/MacOS/Xcode```

![](/img/chromium_ios_debug/project.jpg)

但是工程设置仍然有问题，手工尝试了修补，最后还是不行。

![](/img/chromium_ios_debug/miss_info_plist.jpg)

![](/img/chromium_ios_debug/bundle_error.jpg)

结合这些信息，最后只能如下来调试了。

> ```sudo /Applications/Xcode.app/Contents/MacOS/Xcode```

> ```./iossim -d 'iPad Air' Chromium.app```

> ```XCode``` 中 ```Attache``` 上去

![](/img/chromium_ios_debug/attached.jpg)

好吧，暂时只能先这样了。



