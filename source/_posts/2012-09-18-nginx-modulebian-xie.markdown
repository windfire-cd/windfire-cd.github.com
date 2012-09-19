---
layout: post
title: "Nginx Module编写"
date: 2012-09-18 11:13
comments: true
categories: nginx
---

##模块简介

nginx 的模块在源码中对应着是*ngx_module_t*结构的变量，
有一个全局的*ngx_module_t*指针数组，这个指针数组包含了当前编译版本支持的所有模块，
这个指针数组的定义是在自动脚本生成的*objs/ngx_modules.c*文件中

nginx启动的过程:

nginx是一个master主进程＋多个worker子进程的工作模式 ,nginx主进程启动的过程中会按照初始化master、初始化模块、初始化工作进程、（
初始化线程、退出线程）、 退出工作进程、退出master顺序进行，而在这些子过程内部和子过程之间，又会有读取配置、创建配置、
初始化配置、合并配置、http解析、http过 滤、http输出、http代理等过程，在这些过程开始前后、过程中、结束前后等时机，
nginx调用合适的模块接口完成特定的任务

所 谓的合适模块接口，是各个模块通过一些方式注册到系统内的回调函数，这些回调函数都要符合一定的接口规范

##Hello World

##参考

[nginx](http://nginx.org/)

[nginx源码分析](http://blog.csdn.net/kenbinzhang/article/category/603177)

[nginx模块开发](http://cjhust.blog.163.com/blog/#m=0&t=1&c=fks_084064081084083067086086094095085081084075086081087070083)

[解剖Nginx·模块开发篇](http://blog.csdn.net/poechant/article/details/7627828)

[Nginx模块开发指南中文版](http://www.oschina.net/question/12_4180)

[Emiller's Guide To Nginx Module Development](http://www.evanmiller.org/nginx-modules-guide.html)

[ nginx 源码学习笔记](http://blog.csdn.net/lengzijian/article/details/7598996)
