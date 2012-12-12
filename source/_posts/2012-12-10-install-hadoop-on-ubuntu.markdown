---
layout: post
title: "Install Hadoop on Ubuntu"
date: 2012-12-10 18:19
comments: true
categories: hadoop
---

## 概述

Hadoop 由 Apache Software Foundation 公司于 2005 年秋天作为 Lucene 的子项目 Nutch 的一部分正式引入。它受到最先由 Google Lab 开发的 MapReduce 和 Google File System 的启发。2006 年 3 月份，MapReduce 和 Nutch Distributed File System (NDFS) 分别被纳入称为 Hadoop 的项目中。
Hadoop 是最受欢迎的在 Internet 上对搜索关键字进行内容分类的工具，但它也可以解决许多要求极大伸缩性的问题

## 安装

### 软件

软件版本分别为

- **java**: *jdk-7u9-linux-x64*
- **hadooop**: *hadoop-1.1.1.tar.gz*
- **eclipse**: *eclipse-java-juno-SR1-linux-gtk-x86_64*

### 步骤

#### JAVA

解压*jdk-7u9-linux-x64.tar.gz*到*/usr/lib/jvm*中

运行

```bash
sudo update-java-alternatives java
```
选择对应的版本

类似配置javac, javaws

完成后判断java以及javac版本

```bash
java -version
javac -version
```

在*/etc/profile*中添加

```bash
export JAVA_HOME=/usr/lib/jvm/jdk1.7.0_09/
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
```

完成后需要重启

#### Hadoop

**创建用户组以及用户hadoop**

```bash
sudo addgroup hadoop
sudo adduser --ingroup hadoop hadoop
```

将hadoop用户组添加进sudoer中，在*/etc/sudoers*中添加如下内容
```bash
%hadoop ALL=(ALL) ALL
```

重启后以hadoop用户登录

**配置ssh**

安装opensssh-server
```bash
sudo apt-get install openssh-server
```

配置密码

```bash
ssh-keygen -t rsa -P ""
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

测试
```bash
ssh localhost
```

可以不用密码登录

**禁用ipv6**
可能出现问题：在ubuntu上使用IPV6会有一个问题，就是不同的网络环境配置hadoop会导致hadoop与IPV6地址绑定

在*/etc/sysctl.conf*中添加
```bash
#disable ipv6
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

完成后重启

测试是否有效
```bash
cat /proc/sys/net/ipv6/conf/all/disable_ipv6
```
0表示没有成功，1表示设置成功

#### 配置hadoop

以single node为例，hadoop版本为1.1.1

解压到某个文件夹内，比如/opt
在*~/.bashrc*中添加
```bash
export HADOOP_INSTALL=/opt
export PATH=$PATH:$HADOOP_INSTALL/bin
```

设置*conf/hadoop-env.sh*，添加
```bash
export JAVA_HOME=/usr/lib/jvm/jdk1.7.0_09/
```

根据不同的启动方式配置相关文件，一共有三种启动方式：本地模式，伪分布式，完全分布式

格式化hdfs
```bash
hadoop namenode -format
```

查看日志


#### Eclipse

## 参考


