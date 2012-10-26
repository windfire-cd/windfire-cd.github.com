---
layout: post
title: "Apache Trafficserver介绍"
date: 2012-10-25 10:54
comments: true
categories: Apache Traffic Server
---

##简介

[Apache Traffic Server][1]可以加速internet访问，加强网站性能并提供良好的
网站托管能力

对于网络上日益增长的数据交流，[Apache Traffic Server][1]作为一个高性能的web
代理缓存服务器，可以通过缓存多次访问以及临近数据应对上述问题。它充分利用网络
带宽，提高内容分发系统性能

##角色

ASP可以作为下述角色使用

- web代理缓存
- 反向代理
- 多级缓存


##模块

Traffic Server可以由下面几个模块组合工作

###The Traffic Server Cache

由一个叫做"对象仓库"的高速对象数据库组成。对象索引是由url或者头信息生成的
可以根据语言以及编码管理对象，对于小文件以及大文件同样高效。一旦配置好的空间
不够用则自动删除较老的数据。其被设计对于磁盘的故障是可以容忍的

###The RAM Cache

被访问最多的对象存储在RAM上，降低磁盘负载。

###The Host Database

保存所有数据源访问dns信息

###The DNS Resolver

一个快速，异步的DNS解析器

##Traffic Server Processes

Traffic Server包含三个工作进程处理用户请求以及进行管理/控制/监控整个系统

**traffic_server**

负责接收网络链接，解析协议，从原始服务器或者缓存中获取数据

**traffic_manager**

负责查看，监控以及重新设置*traffic_server*的属性。还负责自动端口分配，统计
信息，集群管理以及虚拟ip管理

**traffic_cop**

负责监控*traffic_server*以及*traffic_manager*的健康信息，通过定时心跳来收集
两者信息并同步到网页上，一旦两者出现异常，它还负责重启两者


{% img http://trafficserver.apache.org/images/admin/process.jpg %}


##参考

[1]: http://trafficserver.apache.org/ "Apache Traffic Server"

[2]: http://baike.baidu.com/view/8691996.htm "Baidu百科"
