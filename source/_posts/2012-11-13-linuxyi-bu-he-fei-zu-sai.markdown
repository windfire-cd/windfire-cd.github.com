---
layout: post
title: "linux异步和非阻塞"
date: 2012-11-13 18:45
comments: true
categories: 
---

##概念

对于linux来说，异步和非阻塞是两个概念。可以概述如下

- 异步: 使得拥有文件指针的进程或者进程组能够立刻收到内核的SIGIO信号
- 非阻塞: 使得对于文件指针的读写操作不会阻塞于buffer为空的情况


##区别及使用说明

ioctl和FIOASYNC等价于fcntl和*O_ASYNC*。

ioctl和FIONBIO等价于fcntl和*O_NONBLOCK*。

下面两个是等价的

```c
fcntl(socket, F_SETFL, fcntl(s, F_GETFL) | O_NONBLOCK);

nb = 1;
ioctl(s, FIONBIO, &nb);
```

FIOASYNC设置*O_ASYNC*标记，该标记决定fd可以IO时进程是否会收到SIGIO和SIGPOLL信号。

FIONBIO设置*O_NONBLOCK*标记，该标记会改变read，write和同类函数的行为，使得在fd还不能IO时立即返回而不是hang住。

后者经常跟select，poll等函数一起使用，使得主程序不会因为个别socket而影响其他。

一般来说使用select和poll结合非阻塞的文件指针可以对应大部分情况，但是某些时候
需要使用异步的文件指针。比如：如果一个函数处理数据，但是处理时间很长，在其处理的时候
我们需要运行这个函数的进程及时响应网络事件或者内核信号，这时就需要将其置为异步

对于socket来说，如果需要设置异步的话需要三个步骤

1. 必须注册一个响应SIGIO的信号回调函数
2. 通过fcntl设置*F_SETOWN*,使得socket属于某个进程
3. 通过fcntl设置*O——ASYNC*将该socket设置为异步


##非阻塞例子

###设置非阻塞
```c async
signal(SIGIO, &input_handler); /* dummy sample; sigaction(  ) is better */
fcntl(STDIN_FILENO, F_SETOWN, getpid(  ));
oflags = fcntl(STDIN_FILENO, F_GETFL);
fcntl(STDIN_FILENO, F_SETFL, oflags | FASYNC);
```

###网络非阻塞例子

```c recv
#include <stdio.h>  
#include <stdlib.h>  
#include <unistd.h>  
#include <fcntl.h>  
#include <errno.h>  
#include <netinet/in.h>       /*socket address struct*/  
#include <arpa/inet.h>            /*host to network convertion*/  
#include <sys/socket.h>  
#include <sys/types.h>  
#include <signal.h>  
#include <sys/ioctl.h>  
#define MAX_TRANSPORT_LENTH 512  

static int g_var = 0;  
static int g_skt = 0;  
void sig_handler(int signum)  
{  
	char  buf[MAX_TRANSPORT_LENTH+1] = "";  
	int len = 0;  
	len = read(g_skt,buf,MAX_TRANSPORT_LENTH);  
	if (len<0)  
	{  
		perror("Read socket failed");  
		exit(-1);     
	}  
	else  
	{  
		printf("In SIGIO handler,got msg:%s\n",buf);  
	}     
}   

int main()  
{  
	struct sockaddr_in addr;  
	memset(&addr,0,sizeof(addr));  
	addr.sin_family =  AF_INET;  
	addr.sin_addr.s_addr = INADDR_ANY;  
	addr.sin_port = htons(50001);  

	signal(SIGIO,sig_handler);  

	g_skt = socket(AF_INET,SOCK_DGRAM,0);  
	if(g_skt == -1)  
	{  
		perror("Create socket failed");  
		exit(-1);  
	}  

	int len = sizeof(addr);  
	int ret = 0;  

	int on = 1;  
	ret = fcntl(g_skt, F_SETOWN, getpid());//Set process or process group ID to receive SIGIO signals  
	if(-1 == ret)  
	{  
		perror("Fcntl F_SETOWN failed");  
		exit(-1);         
	}  
	ret = ioctl(g_skt, FIOASYNC, &on);  
	if(-1 == ret)      
	{  
		perror("Fcntl FIOASYNC failed");  
		exit(-1);         
	}      
	ret = ioctl(g_skt, FIONBIO, &on);  
	if(-1 == ret)      
	{  
		perror("ioctl FIONBIO failed");  
		exit(-1);         
	}  

	ret = bind(g_skt,(struct sockaddr *)&addr,sizeof(addr));  
	if(-1 == ret)  
	{  
		perror("Bind socket failed");  
		exit(-1);  
	}             
	while(1)  
	{  
		printf("I am running\n");  
		sleep(2);         
	}     
	close(g_skt);     
} 
```

```c send
#include <stdio.h>  
#include <stdlib.h>  
#include <unistd.h>  
#include <fcntl.h>  
#include <errno.h>  
#include <netinet/in.h>       /*socket address struct*/  
#include <arpa/inet.h>            /*host to network convertion*/  
#include <sys/socket.h>  
#include <signal.h>  
#define MAX_TRANSPORT_LENTH 512  

int main()  
{  
	struct sockaddr_in addr;  
	memset(&addr,0,sizeof(addr));  
	addr.sin_family =  AF_INET;  
	addr.sin_addr.s_addr = inet_addr("192.168.1.106");  
	addr.sin_port = htons(50001);  

	int sock;  
	sock = socket(AF_INET,SOCK_DGRAM,0);  
	if(sock == -1)  
	{  
		perror("Create socket failed");  
		exit(-1);  
	}  

	int ret;  
	ret = connect(sock,(struct sockaddr *)&addr,sizeof(addr));  
	if(ret == -1)  
	{  
		perror("Connect socket failed");  
		exit(-1);         
	}         
	while(1)  
	{  
		printf("Will send messge to server\n");  
		write(sock,"Some unknown infomation\n",MAX_TRANSPORT_LENTH);  
		sleep(1);  
	}  

}  
```

##注意

对于tcp的SIGIO来说，很多网络事件发送信号给相应的进程，所以在回调函数内需要进行区分

##参考

1. [unp](http://books.google.com.tw/books?id=ptSC4LpwGA0C&pg=PA664&lpg=PA664&dq=SIGIO+tcp&source=bl&ots=Kr5zTgdmMo&sig=8897At9uxcwSAviR46ExndhqI8U&hl=zh-TW&sa=X&ei=VCSiUN-KGe6gmQWXnoCgDw&ved=0CB8Q6AEwAA#v=onepage&q=SIGIO%20tcp&f=false)
2. [FIOASYNC和FIONBIO的区别是什么？](http://51hired.com/questions/13335/FIOASYNC%E5%92%8CFIONBIO%E7%9A%84%E5%8C%BA%E5%88%AB%E6%98%AF%E4%BB%80%E4%B9%88%EF%BC%9F)
3. [What's the difference between FIONBIO and FIOASYNC for socket?](http://stackoverflow.com/questions/7440571/whats-the-difference-between-fionbio-and-fioasync-for-socket)


