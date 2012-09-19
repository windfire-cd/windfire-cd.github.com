---
layout: post
title: "HBase 安装"
date: 2012-09-19 15:58
comments: true
categories: 
---

##准备

需要安装

- Java sdk并配置*JAVA_HOME*
- Hadoop
- zookeeper

##安装

下载后解压到指定目录

```bash
tar -xvf hbase-0.94.1.tar.gz
cd hbase-0.94.1
```

##配置启动

需要配置hbase保存数据的文件夹，确定用户具有读写权限
编辑*conf/hbase-site.xml*文件

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<property>
<name>hbase.rootdir</name>
<value>file:///DIRECTORY/hbase</value>
</property>
</configuration>
```

其中 *DIRECTORY* 为hbase保存数据的文件夹，默认值为 */tmp/hbase-${user.name}*


启动hbase

```bash
./bin/start-hbase.sh
starting master, logging to /home/hadoop/hbase-0.94.1/bin/../logs/hbase-hadoop-master-localhost.out
```

##测试

```bash
hadoop@localhost:~/hbase-0.94.1$ ./bin/hbase shell
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 0.94.1, r1365210, Tue Jul 24 18:40:10 UTC 2012

hbase(main):001:0> 

```

创建表并插入

```bash
hadoop@localhost:~/hbase-0.94.1$ ./bin/hbase shell
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 0.94.1, r1365210, Tue Jul 24 18:40:10 UTC 2012

hbase(main):001:0> create 'test','cf'
0 row(s) in 4.4750 seconds

hbase(main):002:0> list 'test'
TABLE                                                                                                                                        
test                                                                                                                                         
1 row(s) in 0.1360 seconds

hbase(main):003:0> put 'test', 'row1', 'cf:a', 'value1'
0 row(s) in 0.3430 seconds

hbase(main):004:0> put 'test', 'row2', 'cf:b', 'value2'
0 row(s) in 0.0720 seconds

hbase(main):015:0> put 'test', 'row3', 'cf:c', 'value3'
0 row(s) in 0.0250 seconds

hbase(main):016:0> scan 'test'
ROW                                  COLUMN+CELL                                                                                             
row1                                column=cf:a, timestamp=1348043456231, value=value1                                                      
row2                                column=cf:b, timestamp=1348043476312, value=value2                                                      
row3                                column=cf:c, timestamp=1348043612383, value=value3                                                      
row(s) in 0.1310 seconds

hbase(main):017:0> get 'test', 'row1'
COLUMN                               CELL                                                                                                    
 cf:a                                timestamp=1348043456231, value=value1                                                                   
 1 row(s) in 0.0560 seconds

hbase(main):018:0> disable 'test'
0 row(s) in 2.2520 seconds

hbase(main):019:0> drop 'test'
0 row(s) in 1.6390 seconds

hbase(main):018:0> disable 'test'
0 row(s) in 2.2520 seconds

hbase(main):019:0> drop 'test'
0 row(s) in 1.6390 seconds

hbase(main):020:0> exit
hadoop@localhost:~/hbase-0.94.1$ ./bin/stop-hbase.sh 
stopping hbase.................
hadoop@localhost:~/hbase-0.94.1$ 

```
##备注

可以查看logs中.log结尾的日志，判断启动问题

- **zookeeper连不上**: 可能就没装，或者2181默认端口被占用
- **unable create version file on /home/hbase_data**: 保存数据路径不正确


