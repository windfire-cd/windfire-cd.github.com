---
layout: post
title: "Keepalived使用"
date: 2012-12-24 14:37
comments: true
categories: 
---

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


## 配置文件

## 参考
