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



##参考

[mina_intro_1]: http://blog.csdn.net/wkyb608/article/details/5940004
[develop_by_mina]: http://www.ibm.com/developerworks/cn/opensource/os-cn-apmina/
