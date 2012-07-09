---
layout: post
title: "github.com connection timeout"
date: 2012-07-09 19:41
comments: true
categories: 
---

基于github.com上面进行push, pull操作遇到如下问题

	ssh: connection to host port: 22: Connection timed out lost connection

解决步骤如下：

### 测试ssh

输入:

	ssh -v git@github.com

如果输出如下

		OpenSSH_5.8p1, OpenSSL 1.0.0d 8 Feb 2011
		debug1: Connecting to github.com [207.97.227.239] port 22.
		debug1: connect to address 207.97.227.239 port 22: Connection timed out
		ssh: connect to host github.com port 22: Connection timed out
		ssh: connect to host github.com port 22: Bad file number
	
linux的*Bad file number*是*Timeout*

### 检测ssh端口是否被封

输入

	nmap -sS github.com -p 22

可能看到如下输出：

		Starting Nmap 5.35DC1 ( http://nmap.org ) at 2011-11-05 10:53 CET
		Nmap scan report for github.com (207.97.227.239)
		Host is up (0.10s latency).
		PORT   STATE    SERVICE
		22/tcp ***filtered*** ssh

		Nmap done: 1 IP address (1 host up) scanned in 2.63 seconds
	
如果看到filtered则表明22端口被防火墙封闭了

### 解决问题

在~/.ssh/config(windows是C:\Users\USERNAME\.ssh\config)内添加(config文件没有就创建之)

	Host github.com
	User git
	Hostname ssh.github.com
	PreferredAuthentications publickey
	IdentityFile ~/.ssh/id_rsa
	Port 443

运行：

	ssh -T github.com
