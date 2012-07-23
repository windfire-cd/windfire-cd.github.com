---
layout: post
title: "Apache Mina"
date: 2012-07-12 14:19
comments: true
categories: 
---

##安装配置

##简介


- NIO framework  library,
- client  server framework  library, or
- a networking  socket library.

##简单的时间函数例子

``` java TimeServer

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.charset.Charset;

import org.apache.mina.core.service.IoAcceptor;
import org.apache.mina.core.session.IdleStatus;
import org.apache.mina.filter.codec.ProtocolCodecFilter;
import org.apache.mina.filter.codec.textline.TextLineCodecFactory;
import org.apache.mina.filter.logging.LoggingFilter;
import org.apache.mina.transport.socket.nio.NioSocketAcceptor;

public class MinaTimeServer
{
	private static final int PORT = 9123;

	public static void main( String[] args ) throws IOException
	{
		IoAcceptor acceptor = new NioSocketAcceptor();

		acceptor.getFilterChain().addLast( "logger", new LoggingFilter() );
		acceptor.getFilterChain().addLast( "codec", new ProtocolCodecFilter( new TextLineCodecFactory( Charset.forName( "UTF-8" ))));

		acceptor.setHandler( new TimeServerHandler() );
		acceptor.getSessionConfig().setReadBufferSize( 2048 );
		acceptor.getSessionConfig().setIdleTime( IdleStatus.BOTH_IDLE, 10 );
		acceptor.bind( new InetSocketAddress(PORT) );
	}
}
```

下面是Handler代码

``` java Handler
import java.util.Date;

import org.apache.mina.core.session.IdleStatus;
import org.apache.mina.core.service.IoHandlerAdapter;
import org.apache.mina.core.session.IoSession;

public class TimeServerHandler extends IoHandlerAdapter
{
	@Override
		public void exceptionCaught( IoSession session, Throwable cause ) throws Exception
		{
			cause.printStackTrace();
		}

	@Override
		public void messageReceived( IoSession session, Object message ) throws Exception
		{
			String str = message.toString();
			if( str.trim().equalsIgnoreCase("quit") ) {
				session.close();
				return;
			}

			Date date = new Date();
			session.write( date.toString() );
			System.out.println("Message written...");
		}

	@Override
		public void sessionIdle( IoSession session, IdleStatus status ) throws Exception
		{
			System.out.println( "IDLE " + session.getIdleCount( status ));
		}
}
```

在shell里运行

``` bash 
$telnet 127.0.0.1 9123
```

##日志配置

在工程的src下建立log4j.properties文件，添加代码如下

```
# Set root logger level to DEBUG and its only appender to A1.
log4j.rootLogger=DEBUG, A1

# A1 is set to be a ConsoleAppender.
log4j.appender.A1=org.apache.log4j.ConsoleAppender

# A1 uses PatternLayout.
log4j.appender.A1.layout=org.apache.log4j.PatternLayout
log4j.appender.A1.layout.ConversionPattern=%-4r [%t] %-5p %c{1} %x - %m%n
```

在主函数中添加

``` java

LoggingFilter loggingFilter = new LoggingFilter();
chain.addLast("logging", loggingFilter);         

```

##IoBuffer

替代NIO中的ByteBuffer的，原因如下

- ByteBuffer没有提供fill, get/putString, get/putAsciiInt()等获取放入函数
- ByteBuffer对于变长数据处理不好

>这种解决方式并不佳，Bufer就是Buffer，只是临时存储数据的。并且这种包装不是
>zero-copy的，也许用InputStream
>替代byte buffer是一个更好的办法，相信这些会在Mina 3中改变。

初始化IoBuffer

```java

// Allocates a new buffer with a specific size, defining its type (direct or heap)
public static IoBuffer allocate(int capacity, boolean direct)

// Allocates a new buffer with a specific size
public static IoBuffer allocate(int capacity)

```

创建自增的IoBuffer

>在3.0版本中会被移除，使用InputStream或者固定大小buffer替代

```java

IoBuffer buffer = IoBuffer.allocate(8);
buffer.setAutoExpand(true);

buffer.putString("12345678", encoder);
       
// Add more to this buffer
buffer.put((byte)10);
```

自减空间的buffer

```java

IoBuffer buffer = IoBuffer.allocate(16);
buffer.setAutoShrink(true);
buffer.put((byte)1);
System.out.println("Initial Buffer capacity = "+buffer.capacity());
buffer.shrink();
System.out.println("Initial Buffer capacity after shrink = "+buffer.capacity());

buffer.capacity(32);
System.out.println("Buffer capacity after incrementing capacity to 32 = "+buffer.capacity());
buffer.shrink();
System.out.println("Buffer capacity after shrink= "+buffer.capacity());

```

##IoHandler

Handler处理所有的I/O事件，处于过滤链底部，包含下列函数


- sessionCreated
- sessionOpened
- sessionClosed
- sessionIdle
- exceptionCaught
- messageReceived
- messageSent

**sessionCreated Event**

链接创建触发该事件

**sessionOpened Event**

紧跟在sessionCreated后面触发该事件

**sessionClosed Event**

一旦session关闭，则触发该事件

**sessionIdle Event**

一旦session处于空闲状态触发该事件，udp没有该函数

**exceptionCaught Event**

异常触发该事件，如果是IOException则关闭socket

**messageReceived Event**

一旦接收到消息则触发该事件，大多数应用从这里开始，需要自己处理消息类型

**messageSent Event**

一旦消息响应发送回去触发该事件，发送通过IoSession.write()返回


##参考

[mina_intro_1]: http://blog.csdn.net/wkyb608/article/details/5940004
[develop_by_mina]: http://www.ibm.com/developerworks/cn/opensource/os-cn-apmina/
