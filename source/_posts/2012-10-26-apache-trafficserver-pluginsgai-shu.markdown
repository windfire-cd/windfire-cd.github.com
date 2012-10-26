---
layout: post
title: "Apache Traffic Server Plugins概述"
date: 2012-10-26 14:55
comments: true
categories: Apache Traffic Server
---

##简介

可以利用c语言为ats编写功能插件。ats支持复杂的基于web的处理和缓存模式，其包含一个事件触发的循环

```c
for(;;){
	event = get_next_event();
	handle_event(event);
}
```
你可以编写自己的插件，然后编译成动态库，供ats启动时进行载入。插件注册相应的事件处理函数以及事件类型
一旦ats需要处理某个事件，它会运行所有注册在该事件上的回调函数。

{% img http://trafficserver.apache.org/images/sdk/plugin_process.jpg %}


##用途

插件有以下用途

- HTTP协议处理：过滤，屏蔽，认证用户，传递内容
- 新协议支持：可以支持新的缓存协议

下面是一些插件的例子：

- Blacklisting plugin： 拒绝某些网络访问
- Append transform plugin：将文本通过http响应的content传递回去
- Image conversion plugin：将jpg转化为gif
- Compression plugin：将响应内容进行压缩
- Authorization plugin：认证访问用户
- A plugin that gathers client information：收集请求信息并保存到数据库中
- Protocol plugin：利用ats监听特定端口，处理特殊的协议请求

{% img http://trafficserver.apache.org/images/sdk/Uses.jpg %}

##例子

有如下几个例子

- append-transform.c： 为http响应添加文本信息
- server-transform.c：将请求数据提供给转码服务器进行转码后传回给client端，例如图形转化
- basic-auth.c：http代理认证
- blacklist-1.c：读取配置文件进行代理认证

##插件载入

ats启动后根据plugin.config配置文件决定哪些插件需要进行载入。同时plugin.config被传递给每个插件的初始化函数TSPluginInit。
 records.config确定插件动态库的路径


##配置

下面是一个配置文件例子，包含一个命令行，一个空白行和两个插件配置行

```bash
# This is a comment line.

my-plugin.so junk.example.com trash.example.org garbage.example.edu
some-plugin.so arg1 arg2 $proxy.config.http.cache.on
```
so文件后面是对应的参数序列
插件的配置文件每一行不能大于1023字节，插件的so命名应该是全局唯一的。

- 命令行以#起始
- 空行被葫芦额
- 插件载入顺序和在plugin.config出现的顺序一致

##启动及初始化

每个插件需要定义一个初始化函数TSPluginInit，这个函数读取配置文件并注册对应的事件。

TSPluginInit包含两个参数

- argc: 该插件在plugin.config内的参数个数
- argv: 该插件在plugin.config参数数组


##参考

[Understanding Traffic Server Plugins](http://trafficserver.apache.org/docs/trunk/sdk/getting-started/)

