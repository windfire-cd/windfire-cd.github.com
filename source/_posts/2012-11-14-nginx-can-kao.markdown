---
layout: post
title: "Nginx 参考"
date: 2012-11-14 17:50
comments: true
categories: 
---

##简介

Nginx是当前最流行的HTTP Server之一，根据W3Techs的统计，目前世界排名
（根据Alexa）前100万的网站中，Nginx的占有率为6.8%。与Apache相比，
Nginx在高并发情况下具有巨大的性能优势。
Nginx属于典型的微内核设计，其内核非常简洁和优雅，同时具有非常高的可扩展性。
Nginx最初仅仅主要被用于做反向代理，后来随着HTTP核心的成熟和各种HTTP扩展模块的丰富，
Nginx越来越多被用来取代Apache而单独承担HTTP Server的责任，例如目前淘宝内各个部门正越来越多使用Nginx取代Apache，
据笔者了解，在腾讯和新浪等公司也存在类似情况

##源码

1. [Nginx](www.nginx.org)
2. [Nginx 内存池](http://www.alidata.org/archives/1390)
3. [分析blog 1](http://blog.csdn.net/kenbinzhang/article/category/603177)
4. [osc blog 1](http://my.oschina.net/fqing/blog?catalog=232290)
5. [csdn blog 1](http://blog.csdn.net/dingyujie)

##模块化

1. [Nginx模块开发入门](http://www.codinglabs.org/html/intro-of-nginx-module-development.html)
2. [Emiller's Guide To Nginx Module Development](http://www.evanmiller.org/nginx-modules-guide.html)
3. [Emiller's Advanced Topics In Nginx Module Development](http://www.evanmiller.org/nginx-modules-guide-advanced.html)
4. [subrequest blog ](http://blog.chinaunix.net/uid-17271162-id-3061033.html)
5. [开发nginx模块之Hello World篇](http://www.162cm.com/p/ngx_ext.html)
6. [Nginx开发从入门到精通](http://tengine.taobao.org/book/index.html)
7. [lua with nginx](http://huoding.com/2012/08/31/156)
8. [Nginx+Lua+Redis整合实现高性能API接口](http://blog.latermoon.com/?p=729)

##其他

1. [pagefault blog](http://www.pagefault.info)
2. [csdn blog 2](http://simohayha.iteye.com/)
