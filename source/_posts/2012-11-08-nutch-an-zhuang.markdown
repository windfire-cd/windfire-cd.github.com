---
layout: post
title: "Nutch 安装"
date: 2012-11-08 16:14
comments: true
categories: hadoop zookeeper nutch
---

##简介

Nutch 是一个开源Java 实现的搜索引擎。它提供了我们运行自己的搜索引擎所需的全部工具。包括全文搜索和Web爬虫。

尽管Web搜索是漫游Internet的基本要求, 但是现有web搜索引擎的数目却在下降. 并且这很有可能进一步演变成为一个公司垄断了几乎所有的web搜索为其谋取商业利益.这显然 不利于广大Internet用户.

Nutch为我们提供了这样一个不同的选择. 相对于那些商用的搜索引擎, Nutch作为开放源代码 搜索引擎将会更加透明, 从而更值得大家信赖. 现在所有主要的搜索引擎都采用私有的排序算法, 而不会解释为什么一个网页会排在一个特定的位置. 除此之外, 有的搜索引擎依照网站所付的 费用, 而不是根据它们本身的价值进行排序. 与它们不同, Nucth没有什么需要隐瞒, 也没有 动机去扭曲搜索的结果. Nutch将尽自己最大的努力为用户提供最好的搜索结果

##依赖

[Apache Nutch 2.1][1]2.0版本后将持久化层转化到[Apache Gora][2]上，提供HBase，
Cassandra, Hypertable; Redis ; MySQL, HSQLDB等等。

##安装

**下载后解压**

```bash
tar -xvf apache-nutch-2.1.tar.gz
```

**需要配置属性**

在nutch-site.xml中配置属性
```xml
<property>
 <name>storage.data.store.class</name>
  <value>org.apache.gora.hbase.store.HBaseStore</value>
   <description>Default class for storing data</description>
   </property>
```

去除ivy/ivy.xml中的hbase依赖
```xml
 <!-- Uncomment this to use HBase as Gora backend. -->
     
	     <dependency org="org.apache.gora" name="gora-hbase" rev="0.2" conf="*->default" />
```

在gora.properties添加默认属性
```xml
gora.datastore.default=org.apache.gora.hbase.store.HBaseStore
```


**进入后使用ant进行编译**
```bash
ant
```

##测试

对于2.0版本后，启动前需要保证HBase已经顺利启动，具体步骤见[HBasequick start tutorial][3]

利用下面的命令进行测试

```bash
cd runtime/local/bin
nutch inject ../urls
nutch readdb
```


##问题

**ava.lang.IllegalArgumentException: Not a host:port pair**


##参考

- [1](http://nutch.apache.org/)

- [2](http://gora.apache.org/)
