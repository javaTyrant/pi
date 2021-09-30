# linux的select

在一段指定的时间内，监听用户感兴趣的文件描述符上可读、可写和异常等事件。

## select 机制的优势

为什么会出现select模型？

先看一下下面的这句代码：

int iResult = recv(s, buffer,1024);

这是用来接收数据的，在默认的阻塞模式下的套接字里，recv会阻塞在那里，直到套接字连接上有数据可读，把数据读到buffer里后recv函数才会返回，不然就会一直阻塞在那里。在单线程的程序里出现这种情况会导致主线程（单线程程序里只有一个默认的主线程）被阻塞,这样整个程序被锁死在这里，如果永 远没数据发送过来，那么程序就会被永远锁死。这个问题可以用多线程解决，但是在有多个套接字连接的情况下，这不是一个好的选择，扩展性很差。

int iResult = ioctlsocket(s, FIOBIO, (unsigned long *)&ul); iResult = recv(s, buffer,1024);

这一次recv的调用不管套接字连接上有没有数据可以接收都会马上返回。原因就在于我们用ioctlsocket把套接字设置为非阻塞模式了。不过你跟踪一下就会发现，在没有数据的情况下，recv确实是马上返回了，但是也返回了一个错误：WSAEWOULDBLOCK，意思就是请求的操作没有成功完成。

看到这里很多人可能会说，那么就重复调用recv并检查返回值，直到成功为止，但是这样做效率很成问题，开销太大。

select模型的出现就是为了解决上述问题。
select模型的关键是**使用一种有序的方式，对多个套接字进行统一管理与调度** 。

从流程上来看，使用select函数进行IO请求和同步阻塞模型没有太大的区别，甚至还多了添加监视socket，以及调用select函数的额外操作，效率更差。但是，使用select以后最大的优势是用户可以在一个线程内同时处理多个socket的IO请求。用户可以注册多个socket，然后不断地调用select读取被激活的socket，即可达到在同一个线程内同时处理多个IO请求的目的。而在同步阻塞模型中，必须通过多线程的方式才能达到这个目的。

```java
{
	select(socket);
	while(1) 
	{
		sockets = select();
		for(socket in sockets) 
		{
			if(can_read(socket)) 
			{
				read(socket, buffer);
				process(buffer);
			}
		}
	}
}
```

```java
#include <sys/select.h>
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>
int select(int maxfdp, fd_set *readset, fd_set *writeset, fd_set *exceptset,struct timeval *timeout);
```

maxfdp：被监听的文件描述符的总数，它比所有文件描述符集合中的文件描述符的最大值大1，因为文件描述符是从0开始计数的；

readfds、writefds、exceptset：分别指向可读、可写和异常等事件对应的描述符集合。

timeout:用于设置select函数的超时时间，即告诉内核select等待多长时间之后就放弃等待。timeout == NULL 表示等待无限长的时间

timeval结构体定义如下：

```java
struct timeval
{      
    long tv_sec;   /*秒 */
    long tv_usec;  /*微秒 */   
};
```

返回值：超时返回0;失败返回-1；成功返回大于0的整数，这个整数表示就绪描述符的数目。

以下介绍与select函数相关的常见的几个宏：

```java
#include <sys/select.h>   
int FD_ZERO(int fd, fd_set *fdset);   //一个 fd_set类型变量的所有位都设为 0
int FD_CLR(int fd, fd_set *fdset);  //清除某个位时可以使用
int FD_SET(int fd, fd_set *fd_set);   //设置变量的某个位置位
int FD_ISSET(int fd, fd_set *fdset); //测试某个位是否被置位
```



## select总结：

select本质上是通过设置或者检查存放fd标志位的数据结构来进行下一步处理。这样所带来的缺点是：

1、单个进程可监视的fd数量被限制，即能监听端口的大小有限。一般来说这个数目和系统内存关系很大，具体数目可以cat/proc/sys/fs/file-max察看。32位机默认是1024个。64位机默认是2048.

2、 对socket进行扫描时是线性扫描，即采用轮询的方法，效率较低：当套接字比较多的时候，每次select()都要通过遍历FD_SETSIZE个Socket来完成调度,不管哪个Socket是活跃的,都遍历一遍。这会浪费很多CPU时间。如果能给套接字注册某个回调函数，当他们活跃时，自动完成相关操作，那就避免了轮询，这正是epoll与kqueue做的。

3、需要维护一个用来存放大量fd的数据结构，这样会使得用户空间和内核空间在传递该结构时复制开销大。



# linux的epoll