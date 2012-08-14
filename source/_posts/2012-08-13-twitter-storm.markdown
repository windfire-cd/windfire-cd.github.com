---
layout: post
title: "Twitter Storm"
date: 2012-08-13 10:20
comments: true
categories: 
---

##简介

Twitter Storm是Twitter开源的一个实时流数据处理框架，主要基于Java，部分可用
python。原来是BackType开发的，后被Twitter收购，整理后开源。主要用于下面三个领域

1. 信息流处理(stream processing): 处理实时数据和更新数据库
2. 连续计算(continuous computation): 可以连续查询并反馈给用户
3. 分布式远程过程调用(distributed rpc): 处理密集数据。

特点如下

1. 编程模型类似mapreduce，简化复杂性
2. 使用各种语言，默认支持clojure, java, ruby, python。支持其他语言需要实现storm的通信协议
3. 容错性。管理工作进程和节点的故障
4. 水平扩展。计算在多线程，进程和服务器间进行
5. 可靠消息处理。storm保证每个消息至少被完整处理一次
6. 快速。 使用zeromq为底层消息队列
7. 本地模式。用于快速开发和调试

##参考

[Twitter Storm：What & Why？](http://hitina.lofter.com/post/a8c5e_12e927/)

[Twitter Storm 实时数据处理框架分析总结](http://www.open-open.com/lib/view/open1328286398374.html)

[Twitter Storm 在生产集群运行拓扑](http://chenlx.blog.51cto.com/4096635/748737)

[Twitter Storm blog 参考](http://blog.csdn.net/azhao_dn/article/category/937267)

[blog one](http://blog.csdn.net/larrylgq/article/details/7326058)

[tter storm 配置项 ](http://blog.csdn.net/larrylgq/article/details/7326058)

[storm-starter](https://github.com/nathanmarz/storm-starter)

[storm](https://github.com/nathanmarz/storm/)


##storm-start

###编译及安装

**jdk**

首先需要安装jdk支持否则会提示缺少_tools.jar_，需要注意的是要安装jdk6，jdk7编译
会有问题。具体步骤见[install jdk on ubuntu](http://www.devsniper.com/ubuntu-12-04-install-sun-jdk-6-7/)

*注意如果先安装jdk，然后安装maven2的话，需要重新设置下jdk版本，具体见上面的安装*

**twitter4j**

然后去下载[twitter4j](https://github.com/twitter/twitter4j.git)，否则安装时会
提示缺少这个库支持，而且也下载不来，应该是被墙了。

在编译twitter4j时，利用其文件夹内的package.sh文件。只有twitter-core编译成功
其他的没有库支持，而且也下不来，需要首先安装twitter-core

下载最新的jdk6进行编译的话，缺少json包支持，需要在twitter4j-core文件夹中的
pom.xml中添加json包依赖见[maven2 json](http://mvnrepository.com/artifact/org.json/json/20090211)

需要忽略test进行编译打包，首先完成core的编译，然后安装
```bash
$cd  twitter4j-core
$mvn compile
$mvn package -Dmaven.test.skip=true
$mvn install:install-file -DgroupId=org.twitter4j -DartifactId=twitter4j-core -Dversion=2.2.6-SNAPSHOT -Dpackaging=jar -Dfile=target/twitter4j-core-2.2.6-SNAPSHOT.jar
$cd ..
```

然后编译安装twitter-async
```bash
$cd twitter4j-async
$mvn compile
$mvn package -Dmaven.test.skip=true
$mvn install:install-file -DgroupId=org.twitter4j -DartifactId=twitter4j-async -Dversion=2.2.6-SNAPSHOT -Dpackaging=jar -Dfile=target/twitter4j-async-2.2.6-SNAPSHOT.jar
$cd ..
```

最后编译安装twitter-stream
```bash
$cd twitter4j-stream
$mvn compile
$mvn package -Dmaven.test.skip=true
$mvn install:install-file -DgroupId=org.twitter4j -DartifactId=twitter4j-stream -Dversion=2.2.6-SNAPSHOT -Dpackaging=jar -Dfile=target/twitter4j-stream-2.2.6-SNAPSHOT.jar
$cd ..
```

**storm-start**

利用maven进行编译测试

```bash
mvn -f m2-pom.xml compile exec:java -Dexec.classpathScope=compile -Dexec.mainClass=storm.starter.WordCountTopology
```

打包

```bash
mvn -f m2-pom.xml package
```

