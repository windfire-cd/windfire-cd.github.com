---
layout: post
title: "java设计模式"
date: 2012-07-19 11:23
comments: true
categories: 
---


##Factory Method

1. Product 定义了由factory method创建对象的统一接口
2. ConcreteProduct是具体的类
3. Creator为一般抽象类，声明若干factory method方法
4. ConcreteCreator重载factory method创建某个ConcreteProduct

{% img http://www.ibm.com/developerworks/cn/java/l-factory/fig1.gif %}

下面是JavaMail的结构

{% img http://www.ibm.com/developerworks/cn/java/l-factory/fig2.gif %}

**重用性说明**

{% img http://www.ibm.com/developerworks/cn/java/l-factory/fig3.gif %}


**扩展性说明**

如果有新的邮件协议为NewP,扩展Store为NewPStore, 扩展新类NewPFolder以及NewPMessage


**带参数的模式**

Parameterized Factory

对于Parameterized factory method模式，其factory method有一参数，用于指明需创建的对象的类型，这样一个类的factory method可以创建多种具体类型（ConcreteProduct）的对象，与Factory Method相同的是它所创建的对象都具有同样的接口Product

[Factory Method 模式在 Javamail 中的应用](http://www.ibm.com/developerworks/cn/java/l-factory/)

##Observer模式



