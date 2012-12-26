---
layout: post
title: "Keepalived使用"
date: 2012-12-24 14:37
comments: true
categories: 
---

## 概述

项目中使用redis和自己编写的模块，需要用keepalived进行管理。下面的是相关资料和步骤。

## 安装

### ppa安装

在Ubuntu12.04上进行安装
```bash
sudo apt-get install libssl-dev openssl libpopt-dev
sudo apt-get install keepalived
```

## 配置


### 源码安装

拷贝*/usr/loacl/etc/rc.d/keepalived*进*/etc/init.d*
修改*/etc/init.d/keepalived*三处
```bash
#. /etc/rc.d/init.d/functions  
. /lib/lsb/init-functions
```

```bash
#. /etc/sysconfig/keepalived  
. /usr/etc/sysconfig/keepalived
```

```bash
start() {  
	echo -n $"Starting $prog: "  
#daemon keepalived ${KEEPALIVED_OPTIONS}  
		daemon keepalived start  
		RETVAL=$?  
		echo  
		[ $RETVAL -eq 0 ] && touch /var/lock/subsys/$prog  
}  
```
创建目录
```bash
sudo mkdir -p /var/lock/subsys
```

安装daemon
```bash
sudo apt-get install daemon
```

### ppa安装

```bash
sudo apt-get install keepaliaved
```

## 配置

### 日志配置

查看日志需要安装syslog
```bash
sudo apt-get install sysklogd
```
修改日志显示配置
```bash
sudo touch /var/log/keepalived.log
```
在*/etc/syslog.conf*后添加
```bash
# keepalived
local0.*        /var/log/keepalived.log
```
重启sysklogd

修改*/etc/init.d/keepalived*
```bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
DAEMON=/usr/sbin/keepalived
DAEMON_OPTS="-f /etc/keepalived/keepalived.conf -d -D -S 0" #属性
NAME=keepalived
DESC=keepalived
CONFIG=/etc/keepalived/keepalived.conf
TMPFILES="/tmp/.vrrp /tmp/.healthcheckers"

#includes lsb functions 
. /lib/lsb/init-functions

test -f $CONFIG || exit 0
test -f $DAEMON || exit 0

case "$1" in
start)
log_daemon_msg "Starting $DESC" "$NAME"
for file in $TMPFILES
do
test -e $file && test ! -L $file && rm $file
done
if start-stop-daemon --start --quiet --pidfile /var/run/$NAME.pid \
	   --exec $DAEMON -- $DAEMON_OPTS; then
	   log_end_msg 0
	   else
	   log_end_msg 1
	   fi
	   ;;
```
重启keepalived

## 测试

利用apache2进行测试，首先安装
```bash
sudo apt-get install apache2
```
其配置文件在*/etc/apache2*中，网页文件为*/var/www/index.html*
根据不同的server修改index.html中的内容

**master**

```bash
global_defs
{
	notification_email {
		sss@126.com
	}
}

vrrp_script apache2_check {
	script "/etc/keepalived/apach2_check.sh"
		interval 5
		weight -20
}

vrrp_sync_groups vp1
{
	group {
		vi1
	}
}

vrrp_instance vi1
{
	state                   MASTER
		interface               eth0
		virtual_router_id       51
		priority                100
		advert_int              1
		authentication {
			auth_type       PASS
				auth_pass       1111
		}
	track_script {
		apache2_check
	}

	virtual_ipaddress {
		10.10.5.120
	}
}
```

**slave**
```bash
global_defs
{
	notification_email {
		sss@126.com
	}
}

vrrp_script apache2_check {
	script "/etc/keepalived/apach2_check.sh"
		interval 5
		weight -20
}

vrrp_sync_groups vp1
{
	group {
		vi1
	}
}

vrrp_instance vi1
{
	state                   BACKUP
		interface               eth0
		virtual_router_id       51
		priority                95
		advert_int              1
		authentication {
			auth_type       PASS
				auth_pass       1111
		}
	track_script {
		apache2_check
	}

	virtual_ipaddress {
		10.10.5.120
	}
}
```
脚本文件在*/etc/keepalived*中
```bash
#!/bin/sh

if [ `ps -C apache2 --no-header| wc -l` -eq 0 ]
then
sudo service apache2 start
sleep 3
echo `ps -C apache2 --no-header| wc -l`
if [ `ps -C apache2 --no-header| wc -l` -eq 0 ];then
sudo service keepalived stop
fi
fi
```

分别启动master和slave，利用下面命令查看当前服务的server
```bash
curl 10.10.5.120
```
然后关闭master上面的apache2，仍然用命令查询状态发现已经转移到slave上


## 配置文件

对于项目中的使用来说，需要同时监测redis和自己写的模块(*asio_manager_sh*)其配置脚本如下

### 通用配置

**redis需要修改的配置**
```bash redis.conf
repl-ping-slave-period 3
repl-timeout 10
```

**通用脚本**
```bash util.sh
#!/bin/sh
KEEPALIVED_LOG=/var/log/keepalived.log
REDIS_BIN=/usr/local/bin
	BAT_DIR=/home/cd/bat/ubuntu
check_process()
{
	name=$1


		pid=`ps -ef | grep -v grep | grep "${name}" | sed -n  '1P' | awk '{print $2}'`;

# echo pid is $pid

	if [ -z $pid ]
		then
			return 1
	else
		return 0
			fi

}

check_redis()
{

	ip=$1
		$REDIS_BIN/redis-cli -h $ip ping

		if [ $? -eq 0 ]

			then
				return 0
		else
			return 1

				fi

}
```

**启动redis脚本**

```bash db.sh
#!/bin/sh

. /home/cd/bat/ubuntu/util.sh

$REDIS_BIN/redis-server /home/cd/bat/ubuntu/redis.conf&
```

**启动asio_manager_sh**
```bash run.sh
#!/bin/sh

. /home/cd/bat/ubuntu/util.sh

$BAT_DIR/asio_manager_sh --LocalServerIp=10.10.5.120 --WatchDogProtoPort=5200 --ClientPrivatePort=5000 --RedisServerIP=10.10.5.120 --RedisServerPort=6379 --LogConfigFile=$BAT_DIR/log4cplus.properties &
```

**启动keepalived脚本**
```bash start_keepalived.sh
#!/bin/sh

sudo service keepalived start
```

**停止所有脚本**
```bash stop_all.sh
#!/bin/sh

sudo service keepalived stop
killall asio*
killall redis-server
```

**检测manager脚本**
```bash manager_check.sh
#!/bin/sh

. /home/cd/bat/ubuntu/util.sh

check_process asio_manager_sh

exit $?
```

**检测redis脚本**
```bash redis_check.sh
#!/bin/sh

. /home/cd/bat/ubuntu/util.sh

check_redis 127.0.0.1

exit $?
```

**检测redis和manager脚本**
```bash both_check.sh
#!/bin/sh
. /home/cd/bat/ubuntu/util.sh

check_redis 127.0.0.1

rc_val=$?


check_process asio_manager_sh

mc_val=$?

if [ $rc_val -eq 0 ] && [ $mc_val -eq 0 ]
then
exit 0
else
exit 1
fi
```
**停止keepalived时运行的脚本**
```bash to_stop.sh
#!/bin/sh

. /home/cd/bat/ubuntu/util.sh

sudo echo "Stop redis and manager....." >> $KEEPALIVED_LOG
sudo killall redis-server
sudo killall asio_manager_sh
```

**链接出错脚本**
```bash to_fault.sh
#!/bin/sh

. /home/cd/bat/ubuntu/util.sh

sudo echo "fault detect..." >> $KEEPALIVED_LOG

sudo service keepalived stop
```
### master上脚本

**成为master脚本**
```bash to_master.sh
#!/bin/sh

. /home/cd/bat/ubuntu/util.sh

#sudo echo "sync the data from slave..." >> $KEEPALIVED_LOG

$REDIS_BIN/redis-cli -h 127.0.0.1 -p 6379 slaveof 10.10.5.119 6379 >> $KEEPALIVED_LOG 2>&1

sleep 15

$REDIS_BIN/redis-cli -h 127.0.0.1 -p 6379 slaveof no one >> $KEEPALIVED_LOG 2>&1

$BAT_DIR/run.sh
```

**成为backup脚本**
```bash to_backup.sh
#!/bin/sh

#KEEPALIVED_LOG=/var/log/keepalived.log
#REDIS_BIN=/usr/local/bin

. /home/cd/bat/ubuntu/util.sh

echo "sync data from slave...." >> $KEEPALIVED_LOG
#sudo service keepalived stop

sudo killall asio_manager_sh

sleep 5
$REDIS_BIN/redis-cli -h 127.0.0.1 -p 6379 slaveof 10.10.5.119 6379 >> $KEEPALIVED_LOG 2>&1
sleep 15
```


**keepalived配置文件**
```bash keepalived.conf
global_defs
{
	        notification_email {
				                sss@126.com
								        }
}

vrrp_script apache2_check {
	script "/etc/keepalived/apach2_check.sh"
	interval 5
	weight 20
}

vrrp_script manager_check {
	script "/home/cd/bat/ubuntu/manager_check.sh"
	interval 5
	weight 20
}

vrrp_script redis_check {
	script "/home/cd/bat/ubuntu/redis_check.sh"
	interval 5
	weight 20
}

vrrp_script both_check {
	script "/home/cd/bat/ubuntu/both_check.sh"
	interval 5
	weight -20
}

vrrp_instance vi1
{
	state                   MASTER
		interface               eth0
		virtual_router_id       51
		priority                100
		advert_int              1
		authentication {
			auth_type       PASS
				auth_pass       1111
		}


	track_script {
		both_check
	}


	virtual_ipaddress {
		10.10.5.120
	}

	notify_master /home/cd/bat/ubuntu/to_master.sh
		notify_backup /home/cd/bat/ubuntu/to_backup.sh
		notify_stop   /home/cd/bat/ubuntu/to_stop.sh
#       notify_fault  /home/cd/bat/ubuntu/to_fault.sh
}
```

### backup上脚本

**成为master**
```bash to_master.sh
#!/bin/sh

. /home/cd/bat/ubuntu/util.sh

sudo echo "change to master" >> $KEEPALIVED_LOG
#sleep 15

$REDIS_BIN/redis-cli -h 127.0.0.1 -p 6379 slaveof no one >> $KEEPALIVED_LOG 2>&1

#$REDIS_BIN/redis-cli -h 127.0.0.1 -p 6379 slaveof 10.10.5.88 6379 >> $KEEPALIVED_LOG 2>&1

sleep 10

$BAT_DIR/run.sh
```

**成为backup**
```bash to_backup.sh
#!/bin/sh

. /home/cd/bat/ubuntu/util.sh

echo "sync data from slave...." >> $KEEPALIVED_LOG

sudo killall asio_manager_sh

sleep 15

$REDIS_BIN/redis-cli -h 127.0.0.1 -p 6379 slaveof 10.10.5.88 6379 >> $KEEPALIVED_LOG 2>&1
```

**keepalived配置文件**
```bash keepalived.conf
global_defs
{
	        notification_email {
				                sss@126.com
								        }
}

vrrp_script apache2_check {
	script "/etc/keepalived/apach2_check.sh"
	interval 5
	weight 20
}

vrrp_script manager_check {
	script "/home/cd/bat/ubuntu/manager_check.sh"
	interval 5
	weight 20
}

vrrp_script redis_check {
	script "/home/cd/bat/ubuntu/redis_check.sh"
	interval 5
	weight -20
}

vrrp_script both_check {
	script "/home/cd/bat/ubuntu/both_check.sh"
	interval 5
	weight -20
}

vrrp_instance vi1
{
	state                   BACKUP
		interface               eth0
		virtual_router_id       51
		priority                90
		advert_int              1
		authentication {
			auth_type       PASS
				auth_pass       1111
		}

	track_script {
		redis_check
	}

	virtual_ipaddress {
		10.10.5.120
	}

	notify_master /home/cd/bat/ubuntu/to_master.sh
		notify_backup /home/cd/bat/ubuntu/to_backup.sh
		notify_stop   /home/cd/bat/ubuntu/to_stop.sh
#       notify_fault  /home/cd/bat/ubuntu/to_fault.sh
}
```

**注**:

- 需要首先启动master，然后等待第一次both_check成功后再启动backup

- 先启动redis再启动keepalived,必要的话可以用chkconfig进行启动配置

## 问题

## 参考

1. [通过Keepalived实现Redis Failover自动故障切换功能](http://heylinux.com/archives/1942.html)
2. [keepalived的log](http://www.kongch.com/2011/11/keepalived-log/)
3. [keepalived安装配置](http://blog.csdn.net/zll_liang/article/details/8001260)
4. [*keealived-vrrp_script*](http://ialloc.org/2012/keealived-vrrp_script/)
