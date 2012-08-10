---
layout: post
title: "Zikpin Intro"
date: 2012-08-09 11:31
comments: true
categories: 
---

##简介
Zipkin是一个分布式的追踪系统，可以通过该系统获取各个服务的时间数据。Twitter用
它来管理信息收集以及查询服务组件。它的思想来源于[Google Dapper][]

对一个分布式系统进行跟踪监控的原因在于我们可以获取系统服务的深层次数据，比如
某个服务的响应时间，根据这点可以判断当前分布式系统的性能瓶颈以及其他信息。Zikpin
获取这些信息后通过浏览器显示在web上

其体系结构如下

{% img https://github.com/twitter/zipkin/raw/master/doc/architecture-0.png %}

每个被追踪的消息在系统中传送时都带了一个追踪标识，zipkin通过该标识在追踪系统
中标记信息，判断时间。所有带有追踪标识的消息在被服务端传送出去时通过工具库将
该消息传送给zipkin系统。

{ % img https://github.com/twitter/zipkin/raw/master/doc/architecture-1.png % }

##组件

**Finagle**
> 这使一个基于JVM的异步网络框架，通过它可以创建异步RPC，任何基于JVM的语言都
> 可以用它来创建客户端/服务端

*Frinagle*在twitter中被广泛使用。可以在其上添加追踪功能，目前客户端和服务端均
支持Trift和HTTP协议，缓存部分只支持Memcache和Redis(So far we have client/server support for Thrift and HTTP as well as client only support for Memcache and Redis.)

初始化一个finagle server并绑定追踪服务

```java
ServerBuilder()
  .codec(ThriftServerFramedCodec())
    .bindTo(serverAddr)
	  .name("servicename")
	    .tracerFactory(ZipkinTracer())
		  .build(new SomeService.FinagledService(queryService, new TBinaryProtocol.Factory()))
```

初始化一个客户端类似上面的操作，一旦指定Zipkin为追踪服务，那么请求在开始和
结束的时候被自动追踪，相应的服务和机器也会被同时记录。

可以自定义添加更多的追踪信息

```java
Trace.record("starting that extremely expensive computation")
```

除了字符串也可以添加key-value信息

```java
Trace.recordBinary("http.response.code", "500")
```
**Ruby Thrift**

利用一个gem追踪请求，可以用里面的RackHandler生成id，并通知追踪系统该id。下面
是一个ruby thrift client的例子

```ruby
client = ThriftClient.new(SomeService::Client, "127.0.0.1:1234")
client_id = FinagleThrift::ClientId.new(:name => "service_example.sample_environment")
FinagleThrift.enable_tracing!(client, client_id), "service_name")
```
**Querulous**

是一个一个SQL的接口库，基于Scala实现

**Cassie**

是一个Finagle内部的基于Cassandra client库的实现

```java
cluster.keyspace(keyspace).tracerFactory(ZipkinTracer())
```

**Transport**

利用Scribe作为传输模块，利用该模块可以将服务的追踪信息传输给zipkin和hadoop。
> Scribe已经停止开发了，是否有替换？

**Zipkin collector daemon**

信息收集器，一旦追踪信息传送到收集器，该模块会判断其有效性，存储并索引它。

**Storage**

存储模块，现在是利用Cassandra实现。可以选择不同的存储方式
> 比如HBase?

**Zipkin query daemon**

查询模块，通过简单的Trift api查找追踪存储后的信息

**UI**

显示追踪信息，该模块是一个利用D3的Rails应用

模块间关系如下：

{% img https://github.com/twitter/zipkin/raw/master/doc/modules.png%}


##安装配置


###Cassandra

 	利用下面的命令链接当前工程
	```bash
	bin/cassandra-cli -host localhost -port 9160 -f zipkin-server/src/schema/cassandra-schema.txt
	```

###Zookeeper
	
###Scribe

	Scribe配置如下

	```xml
	<store>
	  category=zipkin
	    type=network
		  remote_host=123.123.123.123
		    remote_port=9410
			  use_conn_pool=yes
			    default_max_msg_before_reconnect=50000
				  allowable_delta_before_reconnect=12500
				    must_succeed=no
					</store>
	```
###Zipkin server

```bash

git clone https://github.com/twitter/zipkin.git
cd zipkin
cp zipkin-scribe/config/collector-dev.scala zipkin-scribe/config/collector-prod.scala
cp zipkin-server/config/query-dev.scala zipkin-server/config/query-prod.scala
Modify the configs above as needed. Pay particular attention to ZooKeeper and Cassandra server entries.
bin/sbt update package-dist (This downloads SBT 0.11.2 if it doesn't already exist)
scp dist/zipkin*.zip [server]
ssh [server]
unzip zipkin*.zip
mkdir -p /var/log/zipkin
zipkin-scribe/scripts/collector.sh -f zipkin-scribe/config/collector-prod.scala
zipkin-server/scripts/query.sh -f zipkin-server/config/query-prod.scala
```

###Zipkin UI

是一个标准的Rails 3应用

1. 升级Zookeeper server的配置。用来定位查询器
2. 发布到何时的Rail 3 server上

**zipkin-tracer gem**

通过Rack Handler在Rails 应用中添加追踪信息，在 config.ru中

```ruby
use ZipkinTracer::RackHandler
  run <YOUR_APPLICATION>
```

 
##运行一个简单的Hadoop任务

如果设置Scribe存储到Hadoop上，会产生一系列难以操作数据的报告。因此，利用一个
[Scalding](http://github.com/twitter/scalding)来写Hadoop任务

1. 要运行hadooop任务，需要生成一个全包含的jar包
	
	```
	sbt 'project zipkin-hadoop' compile assemble
	```
2. 修改scald.rb指向需要运行hadoop任务的机器名

3. 如果需要，升级scald.rb中的jarfile的版本

4. 通过scald.rb脚本运行hadoop任务

	```bash
	./scald.rb --hdfs com.twitter.zipkin.hadoop.[classname] --date yyyy-mm-ddThh:mm yyyy-mm-ddThh:mm --output [dir]
	```


##Google论文笔记
[Google Dapper][]是google发表于2010年的一篇关于分布式服务监控以及查询的论文

###目的

###方式

###跟随者

[Google Dapper]: http://research.google.com/pubs/pub36356.html
