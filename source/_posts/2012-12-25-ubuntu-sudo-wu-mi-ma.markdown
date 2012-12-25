---
layout: post
title: "Ubuntu sudo 无密码"
date: 2012-12-25 16:08
comments: true
categories: 
---

## 使得sudo无需密码

需要修改*/etc/sudoer*
尽量使用
```bash
sudo visudo
```
进行修改，保证不会因为出错无法使用sudo
```bash
%sudo ALL=NOPASSWD: ALL
```

## 直接修改*/etc/sudoer*出错处理

进入recovery模式的root账户，执行
```bash
chmod 666 /dev/null
mount -o remount,rw /
```
去除只读模式后，修改*/etc/sudoer*文件
最后
```bash
init 6
```
