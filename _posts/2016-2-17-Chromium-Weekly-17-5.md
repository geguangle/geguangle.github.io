---
layout:     post
title:      "RefPtr Basics"
subtitle:   "Chromium Weekly 17"
date:       2016-6-20
author:     "geguangle"
header-img: "img/about-chromium.jpg"
tags:
    - Chromium
    - Architecture
    - 翻译、整理
---

[RefPtr Basics](https://webkit.org/blog/5381/refptr-basics/)

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


