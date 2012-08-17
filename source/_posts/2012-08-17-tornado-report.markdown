---
layout: post
title: "Tornado Report"
date: 2012-08-17 17:57
comments: true
categories: 
---

##来源

本文来源于阅读[Understanding the code inside Tornado, the asynchronous web server](http://golubenco.org/2009/09/19/understanding-the-code-inside-tornado-the-asynchronous-web-server-powering-friendfeed/)这篇
文章的简单记录


##简介

[Tornado](http://tornadoweb.org/)是一个python写的异步网络框架。最早是由FriendFeed开发的，后被facebook收购后开源。相比于流行的python
网络开发框架django，它更加简单，灵活，可定制化性强，但是学习曲线陡峭。比较类似的框架是web.py


##异步接口

对于server端来说，如果考虑是同步的接口类似下面的

```py
def handler_request(self, request):

    answ = self.remote_server.query(request) # this takes 5 seconds

	    request.write_response(answ)
```

并发性低，当然可以采用多线程（多进程）处理，但是开销也较大。

但是采用异步接口，类似如下代码

```py
def handler_request(self, request):

    self.remote_server.query_async(request, self.response_received)     



	def response_received(self, request, answ):    # this is called 5 seconds later

	    request.write(answ)
```

可以参考pythond的异步网络框架twisted


##源码和安装

可以利用*easy_install*或者*pip*进行安装

```bash
$sudo easy_install tornado
```

或者下载后安装
```bash
git clone http://github.com/facebook/tornado.git
cd tornado
sudo python setup.py install
```

源码包里有demo的例子


##IOLoop

这个模块是框架的核心模块： [ioloop.py](http://github.com/facebook/tornado/blob/master/tornado/ioloop.py#L17)

添加一个socket处理代码如下

```py
def add_handler(self, fd, handler, events):

    """Registers the given handler to receive the given events for fd."""

	    self._handlers[fd] = handler

		    self._impl.register(fd, events | self.ERROR)
```

**_handlers**是一个字典对象，对应于epoll里面注册的socket列表。

**self._impl**或者是[select.epoll()](http://docs.python.org/library/select.html#select.epoll),或者是[select.select](http://docs.python.org/library/select.html#select.select)

```py start
def start(self):

	"""Starts the I/O loop.



	The loop will run until one of the I/O handlers calls stop(), which

	will make the loop stop after the current event iteration completes.

	"""

	self._running = True

	while True:



	[ ... ]



	if not self._running:

	break



	[ ... ]



	try:

event_pairs = self._impl.poll(poll_timeout)

	except Exception, e:

	if e.args == (4, "Interrupted system call"):

	logging.warning("Interrupted system call", exc_info=1)

	continue

	else:

	raise



# Pop one fd at a time from the set of pending fds and run

# its handler. Since that handler may perform actions on

# other file descriptors, there may be reentrant calls to

# this IOLoop that update self._events

self._events.update(event_pairs)

	while self._events:

fd, events = self._events.popitem()

	try:

self._handlers[fd](fd, events)

	except KeyboardInterrupt:

	raise

	except OSError, e:

	if e[0] == errno.EPIPE:

# Happens when the client closes the connection

	pass

	else:

	logging.error("Exception in I/O handler for fd %d",

			fd, exc_info=True)

	except:

	logging.error("Exception in I/O handler for fd %d",

			fd, exc_info=True)

```


关于这个循环的停止，可以通过设置*self._running*为false来解决，如果停止也是由事件触发的话，那么可以用poll的内部机制来解决，
可以注册一个匿名的管道，当需要退出时，向管道写入某些数据，然后触发停止事件，代码如下

```py
def __init__(self, impl=None):



	[...]



	# Create a pipe that we send bogus data to when we want to wake

	# the I/O loop when it is idle

	r, w = os.pipe()

	self._set_nonblocking(r)

	self._set_nonblocking(w)

	self._waker_reader = os.fdopen(r, "r", 0)

	self._waker_writer = os.fdopen(w, "w", 0)

	self.add_handler(r, self._read_waker, self.WRITE)





def _wake(self):

	try:

		self._waker_writer.write("x")

	except IOError:

		pass
```

##定时器

对于IOLoop模块来说实现一个定时器非常简单，利用python的bisect模块实现如下

```py
def add_timeout(self, deadline, callback):

	"""Calls the given callback at the time deadline from the I/O loop."""

	timeout = _Timeout(deadline, callback)

	bisect.insort(self._timeouts, timeout)

	return timeout
```

在IOLoop模块中epoll比select要快速很多。

##IOStream模块

[IOStream](http://github.com/facebook/tornado/blob/master/tornado/iostream.py#L25)提供了非阻塞的socket操作

- *read_util*
- *read_bytes*
- *write*

##HTTP server

组合IOLoop模块以及IOStream模块我们可以实现一个异步的[httpserver.py](http://github.com/facebook/tornado/blob/master/tornado/httpserver.py#L31)

其中*HTTPServer*类只是接收socket然后将其放入IOLoop中。

```py
def listen(self, port, address=""):

    assert not self._socket

	self._socket = socket.(socket.AF_INET, socket.SOCK_STREAM, 0)

	flags = fcntl.fcntl(self._socket.fileno(), fcntl.F_GETFD)

	flags |= fcntl.FD_CLOEXEC

	fcntl.fcntl(self._socket.fileno(), fcntl.F_SETFD, flags)

	self._socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

	self._socket.setblocking(0)

	self._socket.bind((address, port))

	self._socket.listen(128)

	self.io_loop.add_handler(self._socket.fileno(), self._handle_events,

	self.io_loop.READ)
```

其中*_handle_events()*响应相关事件

```py
def _handle_events(self, fd, events):

    while True:

	try:

connection, address = self._socket.accept()

	except socket.error, e:

	if e[0] in (errno.EWOULDBLOCK, errno.EAGAIN):

		return

		raise

		try:

stream = iostream.IOStream(connection, io_loop=self.io_loop)

	HTTPConnection(stream, address, self.request_callback,

			self.no_keep_alive, self.xheaders)

	except:

	logging.error("Error in connection callback", exc_info=True)
```


