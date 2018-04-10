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

一、[VS 2015 as default Windows compiler soon](https://groups.google.com/a/chromium.org/forum/?fromgroups#!searchin/chromium-dev/VS$202015$20as$20default$20Windows$20compiler$20soon/chromium-dev/6Wd16Iqhy9Y/QGsipQesBAAJ)

----------

VS2015即将成为Windows平台的默认编译器，文中提到 LTCG (official)编译速度会提升3倍（dramatically improve build times for LTCG (official) builds - these will build about three times faster!）
短期内，2013和2015都可以编译，随着2015下的稳定，会越来越依赖2015的一些特性，到时2013就不能编译了。

二、[Smooth Scrolling in Chromium](https://docs.google.com/document/d/1JQ6jLy-r7vw_I9s3rtWAIK137fMpF_OFVtfVsgOaoTA/)

----------

实验性功能，通过enable-smooth-scrolling控制。
目的就是为了提高用户体验，在鼠标滚轮、键盘方向键、PageUp、PageDown等情景下，将滚动分成多步进行，给人感觉上更加平滑。
影响：
1.鼠标滚轮滚动一次时，页面中的JavaScript会监听到很多滚动事件，并且适时更新scrollLeft / scrollTop。
2.滚轮事件和完成滚屏之间会有一些延迟。

三、[Announcement: Telemetry project has moved to a new repo](https://groups.google.com/a/chromium.org/forum/?fromgroups#!searchin/chromium-dev/Announcement$3A$20Telemetry$20project$20has$20moved$20to$20a$20new$20repo/chromium-dev/b79F7uB5-_c/eNijQAXbBQAJ)

----------
     Telemetry has been moved into the catapult repo, and can now be found in the chromium
     checkout at src/third_party/catapult/telemetry.

原来位于tools/telemetry，后面将移动到src/third_party/catapult/，并使用单独的repo(catapult)。@许攀，注意这个变化。

四、语义化

----------

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

五、[RefPtr Basics](https://webkit.org/blog/5381/refptr-basics/)

Chromium应该借鉴了Webkit中的很多经验，这篇指针历史的介绍，想必大家都看过了，发现最近有更新，So从历史脉络角度简述一下，对上面的语义化应该有帮助。
Webkit中的许多类都是派生自RefCounted，她们的实例通过ref和deref调整引用计数，控制自个的小生命。
2005年的时候，发现好多内存泄露，都是因为ref和deref不匹配造成的无法释放。感觉这就是必然的，人肉ref和deref，出错不是早晚的事么？
So，引入了智能指针RefPtr，来自动ref和deref。但是这样会带来很多没意义的ref和deref，比如传递实参、返回运算结果，这种操作应该说是太常见了，多到以至于会影响性能。
怎么办？对症来个PassRefPtr，专门用于这种场景，引用计数不用发生变化。
从代码上来看，就是下面这个样子。

Raw Pointers
<pre><code>
class Document {
    ...
    Title* m_title { nullptr };
}
Document::~Document()
{
    if (m_title)
        m_title->deref();
}
void Document::setTitle(Title* title)
{
    if (title)
        title->ref();
    if (m_title)
        m_title->deref();
    m_title = title;
}
</code></pre>

RefPtr

	class Document {
	    ...
	   RefPtr<Title> m_title;
	}
	void Document::setTitle(Title* title)
	{
	    m_title = title;
	}

冗余的ref和def，他称之为reference count churn，也戏称为计数风暴。

    RefPtr<Node> createSpecialNode()  
    {  
      RefPtr<Node> a = new Node;  
      a->setSpecial(true);  
      return a;  
    }  
    RefPtr<Node> b = createSpecialNode();
    
PassRefPtr可以避免引用计数的波动

    PassRefPtr<Node> createSpecialNode()  
    {  
    PassRefPtr<Node> a = new Node;  
    a->setSpecial(true);  
    return a;  
    }  
    RefPtr<Node> b = createSpecialNode(); 
    
PassRefPtr传递时会直接交出所有权，下面的例子会出问题

    static RefPtr<Ring> g_oneRingToRuleThemAll;  
      
    void finish(PassRefPtr<Ring> ring)  
    {  
    g_oneRingToRuleThemAll = ring;  
    ...  
    ring->wear();   //Crash!!!!!
    }
    
ring->wear();会挂掉，所以要“专款专用”，根据语义编码~

    static RefPtr<Ring> g_oneRingToRuleThemAll;  
      
    void finish(PassRefPtr<Ring> prpRing)  
    {  
    RefPtr<Ring> ring = prpRing;  
    g_oneRingToRuleThemAll = ring;  
    ...  
    ring->wear();  
    }  

C++11的到来，提供了Move语意，传参时可以通过智能指针的右值来做到减少计数操作了。最新文档中不再提到PassRefPtr了。但是Blink中，PassRefPtr还是存在的，感觉Webkit的做法并不优雅，智能指针就是管理指针的，最好不要在使用时加花样了，可以内部优化。

    class Document {
    ...
    RefPtr<Title> m_title;
    }
    
    void Document::setTitle(RefPtr<Title>&& title)
    {
    m_title = WTF::move(title);
    }
    
    …
    
    RefPtr<Title> untitledTitle = titleFactory().createUniqueUntitledTitle();
    
    document.setTitle(WTF::move(untitledTitle));


