---
layout: post
title: "Zookeeper Guide"
date: 2012-08-15 15:13
comments: true
categories: 
---

##简介

介绍zookeeper的设计以及模型。提供一个简单的例子以及使用方式


##数据模型 data model

分层的命名空间，每个node都可以有孩子。类似于文件系统中允许节点既可以是文件夹也可以是文件。

###znode

znode保存一个状态数据结构

具体参见Zookeeper Intro中关于znode的介绍。

对于zookeeper文档中的名称说明

- znode：表示一个数据节点
- servers： 表示组成zookeeper集群的机器
- quorum peers：仲裁服务器(啥意思？)
- client：表示一个机器或者一个进程使用zookeeper服务

对于开发者来说，znode是主要接触的部分。

**watches**

client可以作为znode的观察者。znode的变换会触发观察者的响应。具体见下面的介绍

**Data Access**

每个znode上的数据可以被原子读写，具有访问权控制。但是znode主要管理的是协调数据，而不是通用数据
协调数据一般小于1M。如果需要处理大数据，应该用hdfs或者nfs之类的。

**Ephemeral Nodes**

临时节点，跟session的生存周期相同

**Sequence Nodes**

顺序节点，根据父节点的序数递增的创建znode。保证命名唯一

###zookeeper的时间

zookeeper用不同的方式查询时间

- zxid
- version number
- ticks
- real time 只有在写入状态结构数据的时间戳会用，其他时候不用

###状态结构

- czxid
- mzxid
- ctime
- mtime
- version
- cversion
- aversion
- ephemeralOwner
- dataLen
- numChildren


##会话 sessions

对于client来说，链接server时创建session，异常或者主动关闭时删除session。下面是状态变换图

{% img http://zookeeper.apache.org/doc/trunk/images/state_dia.jpg %}

应用程序需要提供给client一个以逗号分割的字符串，被分割的每个部分子串表示了一个zookeeper的ip和port。
例如

 "127.0.0.1:4545" or "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002"

client会挑选一个进行链接，失败的话自动链接下一个。

>3.2.0版本添加了一个目录chroot，作为链接后的跟目录
>例如"127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002/app/a"

##观察者 watcher

所有的获取数据的操作可以设置一个观察者，比如getData(), getChildren(), exists()
有三个关键点

1. 一次性动作。一旦被观察者的数据发生变化，一个watche事件被发送给client，然后变动的数据发送给client，这些在一次动作中完成
2. 发送给client。操作立刻发送给client，但是不保证client收到并重置。发送的动作是异步的，但是server端保证不同的client端收到的
event的顺序是相同的
3. server端设置的数据。zookeeper需要保存两个watches列表：数据列表和子节点列表。setData触发数据列表变动，create和delete触发两个列表
变动

watches列表存储在client链接的server上，便于减轻负担和进行分布式处理。

对于watches来说zookeeper能够保证以下几点

- wathces是有序的。
- client先看到znode的数据变动，然后看到变动的数据
- watch事件的顺序和zookeeper service数据发生变动的顺序是相同的

对于watches需要注意

- watch是一次性的动作。
- 由于网络时延，可能在设置watch的时候，znode数据变动多次。
- 断开链接后watches不再有作用

  
##访问授权 ACL

和unix的文件权限类似，但是不仅仅区分user，group，world

ACL permissions

- CREATE: you can create a child node
- READ: you can get data from a node and list its children.
- WRITE: you can set data for a node
- DELETE: you can delete a child node
- ADMIN: you can set permissions


Builtin ACL Schemes


- world
- auth
- digest
- ip

下面的简单例子

```c
#include <string.h>
#include <errno.h>

#include "zookeeper.h"

static zhandle_t *zh;

/**
 * In this example this method gets the cert for your
 *   environment -- you must provide
 */
char *foo_get_cert_once(char* id) { return 0; }

/** Watcher function -- empty for this example, not something you should
 * do in real code */
void watcher(zhandle_t *zzh, int type, int state, const char *path,
		void *watcherCtx) {}

int main(int argc, char argv) {
	char buffer[512];
	char p[2048];
	char *cert=0;
	char appId[64];

	strcpy(appId, "example.foo_test");
	cert = foo_get_cert_once(appId);
	if(cert!=0) {
		fprintf(stderr,
				"Certificate for appid [%s] is [%s]\n",appId,cert);
		strncpy(p,cert, sizeof(p)-1);
		free(cert);
	} else {
		fprintf(stderr, "Certificate for appid [%s] not found\n",appId);
		strcpy(p, "dummy");
	}

	zoo_set_debug_level(ZOO_LOG_LEVEL_DEBUG);

	zh = zookeeper_init("localhost:3181", watcher, 10000, 0, 0, 0);
	if (!zh) {
		return errno;
	}
	if(zoo_add_auth(zh,"foo",p,strlen(p),0,0)!=ZOK)
		return 2;

	struct ACL CREATE_ONLY_ACL[] = {{ZOO_PERM_CREATE, ZOO_AUTH_IDS}};
	struct ACL_vector CREATE_ONLY = {1, CREATE_ONLY_ACL};
	int rc = zoo_create(zh,"/xyz","value", 5, &CREATE_ONLY, ZOO_EPHEMERAL,
			buffer, sizeof(buffer)-1);

	/** this operation will fail with a ZNOAUTH error */
	int buflen= sizeof(buffer);
	struct Stat stat;
	rc = zoo_get(zh, "/xyz", 0, buffer, &buflen, &stat);
	if (rc) {
		fprintf(stderr, "Error %d for %s\n", rc, __LINE__);
	}

	zookeeper_close(zh);
	return 0;
}
```

##一致性保证

zookeeper性能高，可扩展性强，读写性能高（读比写高）。因为放弃了一部分一致性的要求，client可能在server读取到老的数据。但是可以保证

- 顺序一致性
- 更新原子性
- 单一系统视图
- 可靠性：monotonicity condition in Paxos; 不会因为恢复server而回滚数据
- 及时性：保证一定时间内client获取到最新数据。

利用上述特性可以容易的利用zookeeper接口完成leader election, barriers, queues, and read/write revocable locks 操作

zookeeper不保证下面的特性

- Simultaneously Consistent Cross-Client Views

##语言绑定

可以绑定java和c



##简单操作

###处理错误

java和c都返回错误

*java*

*throwing KeeperException, calling code() on the exception will return the specific error code*

*c*

*returns an error code as defined in the enum ZOO_ERRORS*

###链接zookeeper

###读操作

###写操作

###处理观察事件

###zookeeper处理

###java例子

```java Executor.java
/**
 * A simple example program to use DataMonitor to start and
 * stop executables based on a znode. The program watches the
 * specified znode and saves the data that corresponds to the
 * znode in the filesystem. It also starts the specified program
 * with the specified arguments when the znode exists and kills
 * the program if the znode goes away.
 */
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;

import org.apache.zookeeper.KeeperException;
import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooKeeper;

public class Executor
implements Watcher, Runnable, DataMonitor.DataMonitorListener
{
	String znode;

	DataMonitor dm;

	ZooKeeper zk;

	String filename;

	String exec[];

	Process child;

	public Executor(String hostPort, String znode, String filename,
			String exec[]) throws KeeperException, IOException {
		this.filename = filename;
		this.exec = exec;
		zk = new ZooKeeper(hostPort, 3000, this);
		dm = new DataMonitor(zk, znode, null, this);
	}

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		if (args.length < 4) {
			System.err
				.println("USAGE: Executor hostPort znode filename program [args ...]");
			System.exit(2);
		}
		String hostPort = args[0];
		String znode = args[1];
		String filename = args[2];
		String exec[] = new String[args.length - 3];
		System.arraycopy(args, 3, exec, 0, exec.length);
		try {
			new Executor(hostPort, znode, filename, exec).run();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	/***************************************************************************
	 * We do process any events ourselves, we just need to forward them on.
	 *
	 * @see org.apache.zookeeper.Watcher#process(org.apache.zookeeper.proto.WatcherEvent)
	 */
	public void process(WatchedEvent event) {
		dm.process(event);
	}

	public void run() {
		try {
			synchronized (this) {
				while (!dm.dead) {
					wait();
				}
			}
		} catch (InterruptedException e) {
		}
	}

	public void closing(int rc) {
		synchronized (this) {
			notifyAll();
		}
	}

	static class StreamWriter extends Thread {
		OutputStream os;

		InputStream is;

		StreamWriter(InputStream is, OutputStream os) {
			this.is = is;
			this.os = os;
			start();
		}

		public void run() {
			byte b[] = new byte[80];
			int rc;
			try {
				while ((rc = is.read(b)) > 0) {
					os.write(b, 0, rc);
				}
			} catch (IOException e) {
			}

		}
	}

	public void exists(byte[] data) {
		if (data == null) {
			if (child != null) {
				System.out.println("Killing process");
				child.destroy();
				try {
					child.waitFor();
				} catch (InterruptedException e) {
				}
			}
			child = null;
		} else {
			if (child != null) {
				System.out.println("Stopping child");
				child.destroy();
				try {
					child.waitFor();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
			try {
				FileOutputStream fos = new FileOutputStream(filename);
				fos.write(data);
				fos.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
			try {
				System.out.println("Starting child");
				child = Runtime.getRuntime().exec(exec);
				new StreamWriter(child.getInputStream(), System.out);
				new StreamWriter(child.getErrorStream(), System.err);
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}
}
```

```java DataMonitor.java
/**
 * A simple class that monitors the data and existence of a ZooKeeper
 * node. It uses asynchronous ZooKeeper APIs.
 */
import java.util.Arrays;

import org.apache.zookeeper.KeeperException;
import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooKeeper;
import org.apache.zookeeper.AsyncCallback.StatCallback;
import org.apache.zookeeper.KeeperException.Code;
import org.apache.zookeeper.data.Stat;

public class DataMonitor implements Watcher, StatCallback {

	ZooKeeper zk;

	String znode;

	Watcher chainedWatcher;

	boolean dead;

	DataMonitorListener listener;

	byte prevData[];

	public DataMonitor(ZooKeeper zk, String znode, Watcher chainedWatcher,
			DataMonitorListener listener) {
		this.zk = zk;
		this.znode = znode;
		this.chainedWatcher = chainedWatcher;
		this.listener = listener;
		// Get things started by checking if the node exists. We are going
		// to be completely event driven
		zk.exists(znode, true, this, null);
	}

	/**
	 * Other classes use the DataMonitor by implementing this method
	 */
	public interface DataMonitorListener {
		/**
		 * The existence status of the node has changed.
		 */
		void exists(byte data[]);

		/**
		 * The ZooKeeper session is no longer valid.
		 *
		 * @param rc
		 *                the ZooKeeper reason code
		 */
		void closing(int rc);
	}

	public void process(WatchedEvent event) {
		String path = event.getPath();
		if (event.getType() == Event.EventType.None) {
			// We are are being told that the state of the
			// connection has changed
			switch (event.getState()) {
				case SyncConnected:
					// In this particular example we don't need to do anything
					// here - watches are automatically re-registered with 
					// server and any watches triggered while the client was 
					// disconnected will be delivered (in order of course)
					break;
				case Expired:
					// It's all over
					dead = true;
					listener.closing(KeeperException.Code.SessionExpired);
					break;
			}
		} else {
			if (path != null && path.equals(znode)) {
				// Something has changed on the node, let's find out
				zk.exists(znode, true, this, null);
			}
		}
		if (chainedWatcher != null) {
			chainedWatcher.process(event);
		}
	}

	public void processResult(int rc, String path, Object ctx, Stat stat) {
		boolean exists;
		switch (rc) {
			case Code.Ok:
				exists = true;
				break;
			case Code.NoNode:
				exists = false;
				break;
			case Code.SessionExpired:
			case Code.NoAuth:
				dead = true;
				listener.closing(rc);
				return;
			default:
				// Retry errors
				zk.exists(znode, true, this, null);
				return;
		}

		byte b[] = null;
		if (exists) {
			try {
				b = zk.getData(znode, false, null);
			} catch (KeeperException e) {
				// We don't need to worry about recovering now. The watch
				// callbacks will kick off any exception handling
				e.printStackTrace();
			} catch (InterruptedException e) {
				return;
			}
		}
		if ((b == null && b != prevData)
				|| (b != null && !Arrays.equals(prevData, b))) {
			listener.exists(b);
			prevData = b;
		}
	}
}
```


##参考


