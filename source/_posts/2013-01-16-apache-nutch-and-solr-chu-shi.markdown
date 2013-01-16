---
layout: post
title: "Apache Nutch and Solr 初试"
date: 2013-01-16 18:07
comments: true
categories: 
---

## 简介

Apache Nutch 是一个用JAVA语言编写的开源web爬虫项目。通过使用它，我们能够以一种自动化的方式找到web页面上的超链接，
减少了大量的维护工作,例如检查无用的链接或者创建一个所有访问过搜索页面的副本。讲到这里Apache Solr出现，
Solr是一个开源的全文检索框架，通过solr我们能搜索Nutch访问过的页面。幸运的是，整合Nutch和Solr是十分简单的,例如下面的讲解。

Apache Nutch 支持Solr拆箱即用，使得Nutch 和solr的整合非常简单。同时也去除了遗留的依赖问题：
不必在Apchce tomcat上运行老版本的Nutch web应用程序,也不必基于Lucene进行搜索

## Nutch安装测试

nutch 和 solr 都需要首先安装apache hadoop并设置*JAVA_HOME*, *HADOOP_INSTALL*并将其添加进环境变量中
下载nutch[Download](http://mirror.bit.edu.cn/apache/nutch/)的二进制版本,这里面用的是1.6版本
解压到某文件夹内后进入

修改nutch-site.xml
```xml
<property>
<name>http.agent.name</name>
<value>My Nutch Spider</value>
</property>
```

创建种子文件
```bash
mkdir -p urls
touch urls/seed.txt
echo http://nutch.apache.org/ > urls/seed.txt
```

修改conf/regex-urlfilter.txt
将
```bash
# accept anything else
+.
```
替换为
```bash
+^http://([a-z0-9]*\.)*nutch.apache.org/
```

创建抓取文件夹
```bash
mkdir mycrawl
chmod 777 mycrawl
```

运行抓取
```bash
bin/nutch crawl urls -dir mycrawl -depth 3 -topN 5
```

## Solr安装测试

## 问题

**org.apache.hadoop.mapred.InvalidInputException: Input path does not exist**

由于抓取文件夹访问异常造成，需要执行创建mycrawl并赋予777权限

## 参考

1. [nutch1.4 + solr3.5 上路](http://blog.csdn.net/i5secs/article/details/7428050)
2. [NutchTutorial](http://wiki.apache.org/nutch/NutchTutorial)
