---
layout:     post
title:      "Smart Pointer"
subtitle:   "Chromium Weekly 17"
date:       2016-6-20
author:     "geguangle"
header-img: "img/about-chromium.jpg"
tags:
    - Chromium
    - Architecture
    - 翻译、整理
---


谁都不喜欢含糊不清、晦涩难懂的表达，报表中的各种数字、变量名字中的a，b，c，111，222等，猜来猜去，大脑蒙圈~

语义化可应用的范围很广，包括但不限于，所有需要见文知意的地方。

今天说一下智能指针提供的语义化，个人愚见。

"啊，XXX，又空指针了！"

"啊，XXX，指针又野了！"

指针，对于C++工程师来说，不亚于洪水猛兽，发起威来，难免让你加加班，甚至熬上个通宵。

常见问题：忘记delete了、多次delete、delete后再次使用。

这些问题往往都是直接使用裸指针，没有对其进行管理，随着逻辑越来越复杂，代码量越来越多，很难一下子看清全貌引发的。这时候大家基本上会祭出智能指针来驯服他们，将裸指针装入笼子。这里将智能指针视为一个工具，用来管理对象的生命周期。

如果从语义角度看呢？各种智能指针又在表达什么呢？在一个“全裸”时代，裸指针出现在函数参数中、函数返回值中、成员变量中，她就在那里裸着，没有表达任何含义，谁负责new，谁负责delete，只能一路查代码搞清楚。过了许久，再次见到，还是惶惶不安，查代码来压压惊。引入智能指针后就不一样了，所有的指针都被管理起来了，相对而言裸指针也就有意义了，你不需要关心何处new，何时delete，使用就行了，她的人生有人照看。

当然我说的是理想状态，现实还是有点不一样。但这应该是智能指针所应该起到的见文知意的作用。

下面看一些具体例子

    scoped_ptr<net::ServerSocket> server_socket_;
    
用于成员变量时，自己的人生就托付给了她的使用者，不管是外面传递进来的，还是使用者自己new出来的，这真是不求同年同月同日生，但求同年同月同日死了。如果没有生呢？也就无所谓死了，此时用 scoped_ptr照样有意义，请看tabs.h，

    struct CreateProperties {
    CreateProperties();
    ~CreateProperties();

    // Whether the tab should become the active tab in the window. Does not affect
    // whether the window is focused (see $(ref:windows.update)). Defaults to
    // <var>true</var>.
    scoped_ptr<bool> active;

这里是用于扩展的接口部分，用于表达接口中的optional，为空时，说明使用方没有设置此参数，则使用默认值。对应的tabs.json

    "active": {
      "type": "boolean",
      "optional": true,
      "description": "Whether the tab should become the active tab in the window. Does not affect whether the window is 
                   focused (see $(ref:windows.update)). Defaults to <var>true</var>."
    },
    
通过scoped_ptr表达了3个语义：null、true、false。（使用方未设置，使用方设置true，使用方设置false）

用于函数返回值时，调用方接管了其生命周期。
    scoped_ptr<ScrollAndScaleSet> ProcessScrollDeltas();

用于函数参数时，接收方接管了其生命周期。
    void AddAnimation(scoped_ptr<Animation> animation);
如果一个对象从出生到消亡都被scoped_ptr呵护着，那她的生命是可追溯的，是明确的，也是安全的。

有的时候，一个对象要被多个地方使用，那她的消亡时间就不好确定了，何时delete成了问题。
scoped_refptr挺身而出，使用前先增加引用计数，结束使用减少引用计数，保证大家使用的时候都可以用，大家都使用完后自动delete。
所以scoped_refptr出现的地方，都在提示，不是你一个人在使用。

scoped_refptr是对象的一种强引用，很容易导致循环引用，互不放手，谁也不能“回家睡觉”。相应的一定会有弱引用，WeakPtr来打破这种循环。

bind的时候涉及到对象时，如何保证task运行的的时候，对象还在，还活着呢。

A.通过引用计数，bind的时候对其引用计数加1。但是task多了，飞来飞去的时候，这种不确定性不利于排查问题。

B.合理安排对象生命周期，保证task运行时，一定还活着，这时候就不需要通过引用计数来延长自己的生命了。

    favicon_downloader_.reset(new FaviconDownloader(
        web_contents_.get(), urls_to_download_,
        base::Bind(&BookmarkAppInstaller::OnIconsDownloaded,
                    base::Unretained(this))));
                    
Unretained的语义就是不需要增加引用计数，在Object-C中，retain用于增加对象引用计数，release 是用于减少对象的引用计数。
C.对象不在有效时，就不运行task了。传递WeakPtr，有就用，没有就拉到。

    base::Closure RunLoop::QuitClosure() {
      return base::Bind(&RunLoop::Quit, weak_factory_.GetWeakPtr());
    }
    
回调时，会进行检测，

    template <typename Runnable, typename BoundWeakPtr, typename... Args>
    struct InvokeHelper<true, void, Runnable, TypeList<BoundWeakPtr, Args...>> {
      static void MakeItSo(Runnable runnable, BoundWeakPtr weak_ptr, Args... args) {
        if (!weak_ptr.get()) {
          return;
        }
        runnable.Run(weak_ptr.get(), CallbackForward(args)...);
      }
    };
    
方式A，已经不推荐了，尽量不要使用。
一定要运行task，合理安排生命周期，采用方式B。
task的运行与否无所谓，采用方式C。

    Note that too much of our existing code uses refcounting, so just because you see existing
    code doing it does not mean it's the right solution. 
    (Bonus points if you're able to clean up such cases.)
    
scoped_ptr表达的是所有权，拥有对象的“生”杀大权（较真说，“生”是外部完成的）。
scoped_refptr表达的是使用权，要使用就要对象活着，间接影响着对象的生命周期，何时消亡由所有使用者决定，是不确定的。引用计数的方式都有这个问题，生命周期不明晰。
WeakPtr表达的是观察权，我只是看看，就这样。
围绕智能指针出现的Passed、Owned，都是用来更加清晰的表达对象生命周期的“起承转合”。
举个不太恰当的例子，
scoped_ptr好比电影票，只能一人一张。
scoped_refptr好比电影院，只有曲终人散，结束放映，才能关门。
WeakPtr只能在街边看看电影院外面的大屏幕了。


