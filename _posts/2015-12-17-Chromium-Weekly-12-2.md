---
layout:     post
title:      "Chrome WebUI + Mojo proposal"
subtitle:   "Chromium Weekly 12"
date:       2015-12-17
author:     "geguangle"
header-img: "img/about-chromium.jpg"
tags:
    - Chromium
    - Architecture
    - 翻译、整理
---

`目标：`
通过Mojo框架，改善WebUI中JavaScript和C++之间的交互

`背景：`

1.WebUI通过chrome.send发送请求，实现在WebUIExtension::Send中。

2.C++通过WebUI::CallJavascriptFunction回调WebUI中的函数。
chrome.send没有返回值，导致难以跟踪状态，调用方甚至不知道C++运行完毕后，会回调哪个函数。经常需要和C++开发确认相关函数，这点深有体会~
文中举得例子，更是令人发指，那么多函数，如何配套，不看代码真是难以弄清楚。
```
// Bind global handlers
global.setInitialSettings = this.onSetInitialSettings_.bind(this);
global.setUseCloudPrint = this.onSetUseCloudPrint_.bind(this);
global.setPrinters = this.onSetPrinters_.bind(this);
global.updateWithPrinterCapabilities =
    this.onUpdateWithPrinterCapabilities_.bind(this);
global.failedToGetPrinterCapabilities =
    this.onFailedToGetPrinterCapabilities_.bind(this);
global.failedToGetPrivetPrinterCapabilities =
   this.onFailedToGetPrivetPrinterCapabilities_.bind(this);
global.failedToGetExtensionPrinterCapabilities =
    this.onFailedToGetExtensionPrinterCapabilities_.bind(this);
global.reloadPrintersList = this.onReloadPrintersList_.bind(this);
global.printToCloud = this.onPrintToCloud_.bind(this);
```
甚至不同的结果，要调用不同的JS函数~，例子貌似有些极端。

```
void PrintPreviewHandler::OnGotExtensionPrinterInfo(
    const std::string& printer_id,
    const base::DictionaryValue& printer_info) {
 if (printer_info.empty()) {
    web_ui()->CallJavascriptFunction("failedToResolveProvisionalPrinter",
                                     base::StringValue(printer_id));
    return;
  }

  web_ui()->CallJavascriptFunction("onProvisionalPrinterResolved",
                                   base::StringValue(printer_id),
                                   printer_info);
}  
```

`综上，目前的方案有如下缺点`

1.chrome.send不能讲运行结果返回，开发人员必须查看C++代码来确认后续逻辑。反过来也是一样，C++执行完了，到底运行哪个JS函数，也是要查看JS代码，或者更JS开发人员确认。这样看来一个人同时开发C++和JS效率会好些。

2.JS全局命名空间被强烈污染。

3.请求和响应没有直接的关联，如果需求需要做关联，那只能自己在参数里带上一些ID来做自己完成了。

4.各种回调函数名在JS和C++中产生重复，而且还要保持同步，万一对不上，要慢慢找一会的。

5.成功与失败竟然回调不同的函数，更是让人抓狂。

`期望`

1.更好的交互API。

2.自动完成JS和C++的绑定，避免函数名不同步造成的问题。

3.更好的异步API。

4.足够好的性能（但愿性能不要回退）。

`结论：`

1.Mojo上来就满足了了1和2，.mojom将C++和JS的交互展现的清清楚楚，而且再也不担心函数名弄错了。WebUI实现C++接口，JS包含Client APIs即可（就像AMD--Asynchronous Module Definition）。

2.通过Promises实现更好的异步交互。

3.Mojo使交互更加清晰，有利于更好的测试，创建mocks/fakes/更加容易。

4.[omnibox](https://code.google.com/p/chromium/codesearch#chromium/src/chrome/browser/ui/webui/omnibox/)和[site-engagement](https://code.google.com/p/chromium/codesearch#chromium/src/chrome/browser/ui/webui/engagement/site_engagement_ui.cc)已经完成了这个进化

5.[Extension APIs](https://code.google.com/p/chromium/codesearch#chromium/src/extensions/renderer/resources/serial_service.js)可能最终全部使用Mojo实现。

```
define('serial_service', [
    'content/public/renderer/service_provider',
    'data_receiver',
    'data_sender',
    'device/serial/serial.mojom',
    'device/serial/serial_serialization.mojom',
    'mojo/public/js/core',
    'mojo/public/js/router',
    'stash_client',
], function(serviceProvider,
            dataReceiver,
            dataSender,
            serialMojom,
            serialization,
            core,
            routerModule,
            stashClient) { 
```

