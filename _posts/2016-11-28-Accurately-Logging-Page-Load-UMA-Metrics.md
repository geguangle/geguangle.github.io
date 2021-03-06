---
layout:     post
title:      "Page load metrics subsystem"
subtitle:   "Accurately Logging Page Load UMA Metrics"
date:       2016-11-28
author:     "geguangle"
header-img: "img/about-chromium.jpg"
tags:
    - Chromium
    - ToRead
    - 翻译
---


The page load metrics subsystem captures and records UMA metrics related to
page loading. This includes page timing metrics, such as time to first
paint, as well as page interaction metrics, such as number of page loads
aborted before first paint.

The page load metrics subsystem is not supported on iOS, as it requires hooks
into Blink for page timing metrics that are unavailable on iOS.

This component has the following structure:
- chrome/browser/page_load_metrics/: browser process code
- chrome/common/page_load_metrics/: code shared by browser and renderer, such
                                    as IPC messages
- chrome/renderer/page_load_metrics/ : renderer process code

For additional information on this subsystem, see
[Accurately Logging Page Load UMA Metrics](https://docs.google.com/document/d/1HJsJ5W2H_3qRdqPAUgAEo10AF8gXPTXZLUET4X4_sII/edit)


