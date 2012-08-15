---
layout: post
title: "Zookeeper Intro"
date: 2012-08-15 10:48
comments: true
categories: 
---

##简介

[Zookeeper][z1]是一个开源的分布式的协调服务框架，主要为分布式应用程序提供服务。它提供一些列简单的原语进行同步，配置维护，全局命名
等服务。它是java语言实现的，但是可以绑定运行java以及c的程序

它的主要作用是提供协调服务，减轻分布式应用程序的协调负担。

##目标

###简单

对于zookeeper来说，每个分布式节点类似于一个文件系统的文件。它为其提供分层命名空间下的协调通信服务。zookeeper将数据保存在内存中
而不是硬盘中，所以时延较低

###分布式

zookeeper本身就是分布式的

{% img http://zookeeper.apache.org/doc/trunk/images/zkservice.jpg %}

运行在不同机器上的zookeeper服务可以彼此通信。如果大部分的服务器运转正常，则zookeeper依然可用。
>即少部分的机器异常不影响整体服务
每个client都和一个server保持tcp链接
client通过这个链接传递请求和响应以及心跳，一旦server异常，client可以立刻链接别的server

###有序

zookeeper对每次更新提供一个全局序数，随后的操作可以利用该序数进行高层一致性抽象操作，比如同步
>zookeeper提供全局锁，类似google  chuddy？


###快速

zookeeper是一个读取操作比写入操作要快的服务，在上千个服务器上运行zookeeper的话，一般来说，读写速度是10：1
>考虑到分布式的特性，读可以进行分离，但是写的话需要锁以及一致性操作


##架构

{% img http://zookeeper.apache.org/doc/trunk/images/zknamespace.jpg%}


zookeeper的提供一个分层的命名空间。每个节点都可以有孩子节点，类似于文件系统中每个文件也可以作为文件夹。节点被称为znode

znode保存一个包含数据修改版本，ACL修改记录，以及时间戳的状态结构。允许缓存验证和协调更新。每次状态结构修改，则版本自增1.

znode上的数据读取和写入提供原子操作。zookeeper可以允许存在临时节点，节点生存周期和session相同。

zookeeper提供观察功能，client可以作为znode的观察者，一旦znode数据发生变换，那么client会被通知。

##一致性

- 顺序一致性：client的更新会被按照发送顺序操作
- 原子性：更新或者成功或者失败，不会出现其他结果
- 单一的系统链接表示：所有client观察到的系统都是
- 可靠性：一旦更新提交了，在下一个更新来之前都是有效的
- 时效性：对于client来说，看到的都是最新的数据

##API

- create
- delete
- exists
- get data
- set data
- get chirldren
- sync

##实现

下面是zookeeper的结构图

{% img http://zookeeper.apache.org/doc/trunk/images/zkcomponents.jpg %}

在zookeeper服务中，每个zookeeper的节点都有上述几个模块

replicated database是一个内存数据库，保存修改日志到硬盘上。

每个client端，连接一个指定的server提交相关的request。可以从当前server中读取相关信息。

作为协议公认的方式，所有的信息写入都发给一个当前系统固定的server，被称为leader.其他的server被称为follower.其中follower从leader获取
相关消息。zookeeper的消息层负责leader失效的重新选举和follower同步。


##使用及性能

使用zookeeper提供的高层api，可以完成分布式系统中的同步原语，分组以及所有权管理。
其性能测试如下

{% img http://zookeeper.apache.org/doc/trunk/images/zkperfRW-3.2.jpg %}

[Overview](http://zookeeper.apache.org/doc/trunk/zookeeperOver.html)

[z1]: http://zookeeper.apache.org/
