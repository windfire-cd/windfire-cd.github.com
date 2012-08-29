---
layout: post
title: "Nginx源码阅读"
date: 2012-08-29 11:16
comments: true
categories: Nginx Related
---

##简介

nginx [engine x]是Igor Sysoev编写的一个HTTP和反向代理服务器，另外它也可以作为邮件代理服务器。 它从2004开始已经在众多流量很大的俄罗斯网站上使用，包括Yandex、Mail.Ru、VKontakte，以及Rambler。据Netcraft统计，在2011年10月份，世界上最繁忙的网站中有7.84%使用Nginx作为其服务器或者代理服务器。部分成功案例请见：FastMail.FM， Wordpress.com。

Nginx的源码使用的许可为两条款类BSD协议。



- 一个主进程和多个工作进程，工作进程以非特权用户运行；
- 支持的事件机制：*kqueue（FreeBSD 4.1+）、epoll（Linux 2.6+）、rt signals（Linux 2.2.19+）、/dev/poll（Solaris 7 11/99+）、event ports（Solaris 10）、select*以及*poll*；
- 众多支持的kqueue特性包括*EV_CLEAR、EV_DISABLE*（临时禁止事件）、*NOTE_LOWAT、EV_EOF*，可用数据的数量，错误代码；
- 支持*sendfile（FreeBSD 3.1+, Linux 2.2+, Mac OS X 10.5）、sendfile64（Linux 2.4.21+）*和*sendfilev（Solaris 8 7/01+）*；
- 文件*AIO（FreeBSD 4.3+, Linux 2.6.22+）*；
- *Accept-filters（FreeBSD 4.1+）*和 *TCP_DEFER_ACCEPT（Linux 2.4+）*；
- 10000个非活跃的HTTP keep-alive连接仅占用约2.5M内存；
- 尽可能避免数据拷贝操作


##参考

1. [Nginx官网](http://nginx.org/)
2. [nginxsrp](http://code.google.com/p/nginxsrp/wiki/NginxCodeReview)
3. [nginx module from emiller](http://www.evanmiller.org/nginx-modules-guide.html)
4. [csdn blog one](http://blog.csdn.net/livelylittlefish/article/details/6571497)
