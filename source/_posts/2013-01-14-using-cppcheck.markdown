---
layout: post
title: "using cppcheck"
date: 2013-01-14 15:55
comments: true
categories: 
---

## 简介

cppcheck 是一个静态代码检查工具，支持c, c++ 代码；作为编译器的一种补充检查，cppcheck对产品的源代码执行严格的逻辑检查。
 执行的检查包括

1.  自动变量检查
2.  数组的边界检查
3.  class类检查
4.  过期的函数，废弃函数调用检查
5.  异常内存使用，释放检查
6.  内存泄漏检查，主要是通过内存引用指针
7.  操作系统资源释放检查，中断，文件描述符等
8.  异常STL 函数使用检查
9.  代码格式错误，以及性能因素检查

## 安装和使用

```bash
$sudo apt-get install cppcheck
```

使用
```bash
cppcheck -j 4 --enable=all  . 2>err.txt
```


