---
layout: post
title: "Google Python Style"
date: 2013-01-07 13:36
comments: true
categories: 
---

## 概述

对于Python来说，有其自己的一套语法规范，同时由于python在google里的大量应用，类似于google的c++规范。也有一个类似的python style
下面是一些摘要

## 摘要

### 工具

pychecker
```bash
sudo apt-get install pychecker
```

使用的话直接运行
```bash
pychecker test.py
```
只检测文件自身语法，添加only选项
```bash
pychecker --only test.py
```

### 内容

- **使用模块的全路径名来导入每个模块**
- **模块或包应该定义自己的特定域的异常基类, 这个基类应该从内建的Exception类继承**
- **避免全局变量**
- **鼓励使用嵌套/本地/内部类或函数**
- **lambda适用于单行函数**
- **访问和设置数据成员时, 你通常会使用简单, 轻量级的访问和设置函数. 建议用属性来代替它们**
- **永远不要用==或者!=来比较单件, 比如None. 使用is或者is not.**
- **永远不要用==将一个布尔量与false相比较**
- **对于序列(字符串, 列表, 元组), 要注意空序列是false**
- **注意‘0’(字符串)会被当做true**
- **不要依赖内建类型的原子性**
- **避免使用过于技巧性的特性**

## 参考

- [Python语言规范](http://www.bsdmap.com/articles/zh-google-python-style-guide/python_language_rules.html#)
