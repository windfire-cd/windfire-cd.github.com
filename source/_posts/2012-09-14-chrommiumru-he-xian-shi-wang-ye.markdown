---
layout: post
title: "Chrommium如何显示网页"
date: 2012-09-14 14:49
comments: true
categories: chrome
---

##涉及模块

- WebKit: html渲染引擎
- Glue: 将WebKit的对应类型转化为chromium对应类型，被称为"Webkit嵌入层"它是chromium和*test_shell*(可以测试webkit)的基础
- Renderer/Render host: chromium的多进程嵌入层，负责browser进程和render进程间的通知和命令
- WebContent: 方便整合多个沙盒进程中渲染html为一个完整的画面
- TabContentWrapper： 包含一个完整的WebContent实例，并且包含一个插件接口
- Browser：浏览器窗口，包含多个TabContentWrapper

##WebKit

用来渲染web页面，源码位置为*/third_party/WebKit*。WebKit包含一个"WebCore"和一个"JavaScriptCore"，后者主要用来测试，一般使用V8对其
替换。"WebCore"作为核心渲染引擎，在chromium中并不是像safari一样用"WebKit"的原生的接口，只是为了方便称为"WebKit"层。

###WebKit Port

底层包含google实现的基于系统的底层"port"，部分代码并不是平台相关的，可以认为是"WebCore"的一部分，但是比如字体渲染等操作必须用各个
平台自己的方式。

- 网络通信主要是通过chromium的[multi-process resource loading](http://www.chromium.org/developers/design-documents/multi-process-resource-loading)
来实现的，而不是由render进程直接通过系统调用来实现

- 利用来自android的跨平台图形库Skia渲染除字体外的画面。代码位于 */third_party/skia*其入口是*/webkit/port/platform/graphics/GraphicsContextSkia.cpp*


###WebKit Glue

Glue为WebKit的类型和接口提供了一层封装(比如用std::string 代替 WebCore::String)，所有的chromium代码基于glue，这样方便统一类型以及风格
并且减少WebKit的类型以及API变动为chromium带来的问题

"test shell" 提供了一个原生的调用WebKit的方式，它的调用方式和Chromium通过glue调用WebKit完全一致，可以用来进行测试新代码，减少chromium
架构上许多特性，线程或者进程的干扰。它还可以作为WebKit的一个自动化测试工具。但是"test shell" 下行方面和chromium的多进程方式不同，
它在每个shell中单独集成一个"content shell"

##Render进程

{% img http://www.chromium.org/developers/design-documents/displaying-a-web-page-in-chrome/Renderingintherenderer-v2.png %}

render进程基于glue集成webkit port，代码量不大，主要是通过IPC接收主进程的任务。最重要的类是RenderView，位于*/content/renderer/render_view_impl.cc*
类的对象代表了一个web页面，处理来自主进程的浏览命令，该类继承自RenderWidget，RenderWidget提供了绘制和事件处理接口。RenderView通过
render进程中的RenderProcess对象和主进程交互

RenderWidget实现了glue的一个抽象接口WebWidgetDelegate， 对应于 WebCore::Widget。这是一个基础的显示并处理事件的窗口。RenderView继承自
RenderWidget，并且显示一个tab内的内容或者弹出窗口。对于chromium来说RenderWidget不依赖RenderView存在的唯一情况是web页面的选择窗口。

###render线程

每个render包含两个线程，其中render  thread运行RenderView和WebKit代码，还有一个main thread负责IPC。render thread和外部通信，首先将
消息发送给render中的main thread，然后交给browser进程。所以它和browser之间可以是同步的

##Browser进程

{% img http://www.chromium.org/developers/design-documents/displaying-a-web-page-in-chrome/rendering%20browser.png %}

###browser进程的底层处理

browser分为UI线程和IPC线程，IPC线程负责处理browser和render进程间的通信，并且管理外部网络链接。UI线程一旦创建一个RenderProcessHost
则同时在IPC线程内创建一个新的ChannelProxy IPC对象，render通过该对象内的PIPE通道和UI线程进行通信。该对象内存在ResourceMessageFilter可以处理
网络请求。该功能位于ResourceMessageFilter::OnMessageReceived.

UI线程内的RenderProcessHost对象负责给对应的render发送界面相关消息，该功能位于RenderProcessHost::OnMessageReceived.

###browser进程的高层处理

和界面相关的消息大部分在RenderViewHost::OnMessageReceived被处理，剩下的发送给RenderWidgetHost基类，对应于render进程内的RenderView and the RenderWidget
各平台都有自己的显示实现(RenderWidgetHostView[Aura|Gtk|Mac|Win])

在RenderView/Widget上层是WebContents对象，多数消息在这一层被函数响应。一个WebContents对应于一个webpage.它是内容模块的最高层。负责显示
标签内的内容，每个WebContents包含于一个TabContentsWrapper内，对应于chrome内的一个tab。

##举例

###设置光标的流程

**对于render进程来说**

- WebKit内部生成光标设置消息。该消息通过*content/renderer/render_widget.cc*内的RenderWidget::SetCursor被发出
- 调用RenderView内的RenderWidget::Send分配该消息，然后通过RenderThread::Send将消息从render进程发送给browser进程
- 然后调用render进程内的main thread的IPC::SyncChannel，将消息添加到指定的pipe中

**对于browser进程来说**

- RenderProcessHost内的IPC::ChannelProxy通过IPC线程接收该消息。首先通过ResourceMessageFilter过滤是网络消息还是从底层传来的，由于消息
不是网络消息，没有被过滤，被传送给UI线程
- *content/browser/renderer_host/render_process_host_impl.cc*中的RenderProcessHost::OnMessageReceived 接收到该消息，处理几类消息后
将其他消息传送给消息来源的RenderView对应的RenderViewHost。
- *content/browser/renderer_host/render_view_host_impl.cc*内的RenderViewHost::OnMessageReceived接收到该消息，许多消息在此被处理，但是
这个消息会继续被传递给RenderWidget
- *content/browser/renderer_host/render_view_host_impl.cc*存有消息字典以及对应的处理函数RenderWidgetHost::OnMsgSetCursor，然后被特定的
UI函数处理

###鼠标点击的流程

**对于browser进程来说**

- UI线程内的RenderWidgetHostViewWin::OnMouseEvent捕捉到窗口消息，然后调用ForwardMouseEventToRenderer
- 上个函数将事件打包为跨平台的WebMouseEvent，然后将其发送给对应的RenderWidgetHost
- RenderWidgetHost::ForwardInputEvent创建一个IPC消息*ViewMsg_HandleInputEvent*，将WebInputEvent序列化后放入其中，然后调用RenderWidgetHost::Send.
- 接着会调用RenderProcessHost::Send将消息发送给对应的IPC::ChannelProxy.
- IPC::ChannelProxy.会将消息传给browser进程内的IPC线程 ，然后写入对应的pipe中

许多消息在WebContents内创建，比如浏览消息。上面的流程同样适用于这类消息。

**对于render进程来说**

- IPC线程（这里写main thread?）上的IPC::Channel 读取到消息，然后通过代理传送给render线程
- RenderView::OnMessageReceived获取到消息。很多消息在这里直接被处理了。但是这个消息不能在这里处理，被传递给RenderWidget::OnMessageReceived
然后被RenderWidget::OnHandleInputEvent处理
- 通过WebWidgetImpl::HandleInputEvent处理该消息，这个函数代替了WebKit内的PlatformMouseEvent,将事件传递给WebKit内的WebCore::Widget

##参考

- [How Chromium Displays Web Pages](http://www.chromium.org/developers/design-documents/displaying-a-web-page-in-chrome)
- [Inter-process Communication (IPC)](http://www.chromium.org/developers/design-documents/inter-process-communication)
- [Multi-process Resource Loading](http://www.chromium.org/developers/design-documents/multi-process-resource-loading)
- [Content module](http://www.chromium.org/developers/content-module)


