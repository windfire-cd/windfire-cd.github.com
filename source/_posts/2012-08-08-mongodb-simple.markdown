---
layout: post
title: "MongoDB Simple"
date: 2012-08-08 14:23
comments: true
categories: 
---

##简介

- 拓展了关系数据库的功能，比如辅助索引，范围查询以及排序

- 内置MapReduce的支持

- 文档丰富，接口友好

- MongoDB是一个面向文档的数据库。

- 没有模式

- 易于扩展

- 速度快

- 管理方便

##简单安装和运行

1. 直接下载二进制包
2. 在根目录创建/data/db/目录
3. 运行bin/mongod

on Ubuntu

设置GPG key
```bash
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv 7F0CEB10
```

在/etc/apt/sources.list.d/中创建10gen.list添加如下行
```bash
deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen
```

然后运行
```bash
sudo apt-get update
sudo apt-get install mongodb-10gen
```

##设置
配置文件在/etc/mongodb.conf 

##教程
在其官方网站[Mongodb](http://www.mongodb.org/)上有一个Try it out的教程



