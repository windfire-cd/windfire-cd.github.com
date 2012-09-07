---
layout: post
title: "chrome多进程架构阅读"
date: 2012-09-06 17:31
comments: true
categories: chrome
---

##简介

关于Chromium多进程架构综述

##多进程架构解决的问题

很难构建一个不会崩溃和挂起而且绝对安全的渲染器。现代浏览器类似原始的单用户多任务系统
异常操作很容易使得系统崩溃。一个tab错误或者插件错误引起所有的tab崩溃。

对比现代操作系统，其强壮性是通过分离不同任务在不同的进程中来实现的。而且不同用户
只能访问本用户的数据。

所以我们可以用类似的方式来实现浏览器的架构。

##架构概览

Chromium将不同的tabs通过不同的进程处理来保证其崩溃不会影响其他部分。并且严格限制
每个tab对于内存的访问。

运行UI，管理以及插件的进程称为*browser process* or *browser*

每个tab运行的进程称为*render processes* or *renderers*

*renderers*利用*WebKit*解析渲染*HTML*

{% img http://www.chromium.org/developers/design-documents/multi-process-architecture/arch.png?attredirects=0%}

###管理进程

每个渲染进程都有一个全局的*RenderProcess*对象。负责和*browser*交互。*browser*对应保存
一个*RenderProcessHost*管理状态和通信

###管理渲染

每个渲染进程有一个或者多个被*RenderProcess*管理的*RenderView*实例，每个实例对应一个tab
在*browser*里的*RenderProcessHost*持有多个*RenderViewHost*。每个均有不同的ID。browser和特定
tab的通信靠*RenderViewHost*对象来实现，它通过*RenderProcessHost*将消息发送给*RenderProcess*内的*RenderView*

##模块及接口

渲染进程内

- 渲染进程内是*RenderProcess*处理IPC消息，*browser*内是*RenderProcessHost*
- *RenderView*和对应的*RenderViewHost*以及*WebKit*嵌入层进行通信。

浏览进程内

- *Browser*是一个顶层的浏览窗口
- *RenderProcessHost*是浏览进程和渲染进程IPC通信的实例
- *RenderViewHost*封装了和*RenderView*的通信。

细节参见[How Chromium displays web pages ](http://www.chromium.org/developers/design-documents/displaying-a-web-page-in-chrome)

##共享渲染进程

一般来说，每个tab都是在一个新进程中打开。但是某些时候需要多个tab共享一个渲染进程。
比如打开一个需要进行同步操作的web应用（JavaScript中的window.open）

##渲染进程崩溃检测

*browser*进程监控所有IPC链接，一旦发现某个链接断开，那么认为该渲染进程崩溃。目前处理崩溃的方式是显示一个通知崩溃
的页面

##渲染进程沙盒化

因为*WebKit*单独运行在一个进程里，我们可以控制其如何访问以及访问哪些系统资源。比如渲染进程访问网络只能通过主进程进行访问。
另外可以控制其访问用户显示及相关对象。一旦用户打开一个新窗口或者捕捉按将，因为渲染进程都是独立的，那么不会产生错误的显示。

##内存管理

主要在低内存的情况下，提高顶层tab的响应速度。主要通过降低没有顶层tab的*RenderProcess*的*Working set*的大小。提高切换速度。

##参考

[Multi-process Architecture](http://www.chromium.org/developers/design-documents/multi-process-architecture)

