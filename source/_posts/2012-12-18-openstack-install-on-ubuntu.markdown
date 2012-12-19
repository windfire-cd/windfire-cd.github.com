---
layout: post
title: "OpenStack Install On Ubuntu"
date: 2012-12-18 15:54
comments: true
categories: openstack
---

## 概述

对于OpenStack的学习笔记之一，简单记录在Ubuntu 12.04上面安装OpenStack并进行简单
测试。基本步骤按照参考1来进行

## 步骤

### 安装依赖

**ntp服务**

```bash
sudo apt-get install ntp
```

将下面的内容添加进*/etc/ntp.conf*

```bash
server ntp.ubuntu.com iburst
server 127.127.1.0
fudge 127.127.1.0 stratum 10
```

重启ntp服务

```bash
sudo service ntp restart
```

**tgt**

nova-volume需要tgt支持，安装它

```bash
sudo apt-get install tgt
```

然后启动服务

```bash
sudo service tgt start
```

**openiscsi-client**

nova-compute需要

```bash
sudo apt-get install open-iscsi open-iscsi-utils
```

**network**

需要配置网络，如果没有使用NetworkManager，编辑*/etc/network/interfaces*，例如

```bash
auto lo
iface lo inet loopback

auto eth0 
iface eth0 inet static
address 10.42.0.6
network 10.42.0.0
netmask 255.255.255.0
broadcast 10.42.0.255
gateway 10.42.0.1

auto eth1
iface eth1 inet static
address 192.168.22.1
network 192.168.22.0
netmask 255.255.255.0
broadcast 192.168.22.255
```

**bridge-utils**

```bash
sudo apt-get install bridge-utils
sudo /etc/init.d/networking restart
```

**AMQP**

```bash
sudo apt-get install rabbitmq-server memcached python-memcache
```

**kvm**

```bash
sudo apt-get install kvm libvirt-bin
```

### 配置MySQL

```bash
sudo apt-get install -y mysql-server python-mysqldb
```

修改*/etc/mysql/my.cnf*
```bash
bind-address = 0.0.0.0
```

重启mysql
```bash
sudo serivce mysql restart
```

配置数据库
```bash
mysql -u root <<EOF
CREATE DATABASE nova;
GRANT ALL PRIVILEGES ON nova.* TO 'novadbadmin'@'%' 
  IDENTIFIED BY '123456';
EOF

mysql -u root <<EOF
CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glancedbadmin'@'%' 
  IDENTIFIED BY '123456';
EOF

mysql -u root <<EOF
  CREATE DATABASE keystone;
  GRANT ALL PRIVILEGES ON keystone.* TO 'keystonedbadmin'@'%'
    IDENTIFIED BY '123456';
EOF
```

### 安装配置Keystone

```bash
sudo apt-get install keystone python-keystone python-mysqldb python-keystoneclient
```

编辑*/etc/keystone/keystone.conf*
```bash
[sql]
connection = mysql://keystonedbadmin:123456@10.10.5.41/keystone
idle_timeout = 200
```

重启并同步数据库
```bash
sudo service keystone restart
sudo keystone-manage db_sync
```
下载*keystone_data.sh_.txt*和*endpoints.sh_.txt*
修改*keystone_data.sh_.txt*
```bash
ADMIN_PASSWORD=${ADMIN_PASSWORD:-123456}
export SERVICE_TOKEN="ADMIN"
```
运行
```bash
./endpoints.sh -m 10.10.5.41 -u keystonedbadmin -D keystone -p 123456 -K 10.10.5.41 -R RegionOne -E "http://localhost:35357/v2.0" -S 10.10.5.41 -T ADMIN
```

### 安装配置Glance

```bash
sudo apt-get install glance glance-api glance-client glance-common glance-registry python-glance
```
编辑*/ect/glance/glance-api-paste.ini*和*/etc/glance/glance-registry-paste.ini*
```bash
admin_tenant_name = service
admin_user = glance
admin_password = 123456
```

编辑*/ect/glance/glance-registry.conf*
```bash
sql_connection = mysql://glancedbadmin:123456@10.10.5.41/glance
```
在*/etc/glance/glance-registry.conf*和*/etc/glance/glance-api.conf*末尾添加
```bash
[paste_deploy]
flavor = keystone
```

运行
```bash
sudo glance-manage version_control 0
sudo glance-manage db_sync
```

重启
```bash
sudo service glance-api restart && service glance-registry restart
```

下载img
```bash
wget http://uec-images.ubuntu.com/releases/12.04/release/ubuntu-12.04-server-cloudimg-amd64-disk1.img
```



### 安装配置Nova

### 第一个VM

### Dashboard





## 参考

1. [Installing OpenStack Essex (2012.1) on Ubuntu 12.04](http://www.hastexo.com/resources/docs/installing-openstack-essex-20121-ubuntu-1204-precise-pangolin)
