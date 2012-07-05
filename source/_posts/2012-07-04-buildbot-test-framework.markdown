---
layout: post
title: "BuildBot Test Framework"
date: 2012-07-04 13:45
comments: true
categories: 
---

##简介

BuildBot是一个持续集成工具，用于系统的自动化编译，测试。

特点：
- 安装使用简单，但是拓展性良好，并且不限制于特定语言的构建工具
- 多平台支持，如果开发人员在无法方便在其环境中进行测试，可以提交给版本管理工具后很快收到编译及测试结果
- slave端依赖较少，只需要: virtualenv, Python
- slave端可以运行于NAT防火墙后并和Master端进行通信
- 多种状态汇集工具

##安装

##运行

