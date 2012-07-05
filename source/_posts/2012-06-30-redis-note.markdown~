---
layout: post
title: "Redis note"
date: 2012-06-30 02:04
comments: true
categories: 
---

##安装

``` 
$ wget http://redis.googlecode.com/files/redis-2.4.15.tar.gz
$ tar xzf redis-2.4.15.tar.gz
$ cd redis-2.4.15
$ make
```

##简单测试

*run server*

``` 
$src/redis-server
```

*run client*

``` 
$ src/redis-cli
redis> set foo bar
OK
redis> get foo
"bar"
```

##简单教程

这里是一个[在线教程](http://try.redis-db.com/)

###基本数据类型及操作

作为一个key:value类型的nosql,基本的key, value 对应操作如下

- set
- get 

多用户操作统一key对应的value时，简单的原子操作

- incr
- del

数据支持超时操作

- expire
- ttl

###复杂数据类型及操作

[Redis](http://redis.io/) 同其他缓存系统一个较大区别是支持若干复杂数据类型

**list**

有序链表

- rpush
- lpush
- lrange
- lpop
- rpop
- llen

**set** 

无序不重复

- sadd
- sismember
- smembers
- srem
- sunion


**sorted set**

有序集合

- zadd
- zrange

