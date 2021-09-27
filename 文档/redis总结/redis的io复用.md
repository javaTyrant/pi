作者：晨随
链接：https://www.zhihu.com/question/28594409/answer/52763082
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



题主是看redis相关书籍碰到了困惑，那就结合redis源码来回答题主这个问题。
redis源码地址：[antirez/redis · GitHub](https://link.zhihu.com/?target=https%3A//github.com/antirez/redis)

关于I/O多路复用(又被称为“事件驱动”)，首先要理解的是，操作系统为你提供了一个功能，当你的某个socket可读或者可写的时候，它可以给你一个通知。这样当配合非阻塞的socket使用时，只有当系统通知我哪个描述符可读了，我才去执行read操作，可以保证每次read都能读到有效数据而不做纯返回-1和EAGAIN的无用功。写操作类似。操作系统的这个功能通过select/poll/epoll/kqueue之类的系统调用函数来使用，这些函数都可以同时监视多个描述符的读写就绪状况，这样，多个描述符的I/O操作都能在一个线程内并发交替地顺序完成，这就叫I/O多路复用，这里的“复用”指的是复用同一个线程。

以select和tcp socket为例，所谓可读事件，具体的说是指以下事件：
1 socket内核接收缓冲区中的可用字节数大于或等于其低水位SO_RCVLOWAT;
2 socket通信的对方关闭了连接，这个时候在缓冲区里有个文件结束符EOF，此时读操作将返回0；
3 监听socket的backlog队列有已经完成三次握手的连接请求，可以调用accept；
4 socket上有未处理的错误，此时可以用getsockopt来读取和清除该错误。

所谓可写事件，则是指：
1 socket的内核发送缓冲区的可用字节数大于或等于其低水位SO_SNDLOWAIT；
2 socket的写端被关闭，继续写会收到SIGPIPE信号；
3 非阻塞模式下，connect返回之后，发起连接成功或失败；
4  socket上有未处理的错误，此时可以用getsockopt来读取和清除该错误。

Linux环境下，Redis数据库服务器大部分时间以单进程单线程模式运行(执行持久化BGSAVE任务时会开启子进程)，网络部分属于Reactor模式，同步非阻塞模型，即非阻塞的socket文件描述符号加上监控这些描述符的I/O多路复用机制（在Linux下可以使用select/poll/epoll）。服务器运行时主要关注两大类型事件：文件事件和时间事件。文件事件指的是socket文件描述符的读写就绪情况，时间事件分为一次性定时器和周期性定时器。相比nginx和haproxy内置的高精度高性能定时器，redis的定时器机制并不那么先进复杂，它只用了一个链表来管理时间事件，而且目前链表也没有对各个事件的到点时间进行排序，也就是说，每次都要遍历链表检查每个事件是否需要到点执行。个人猜想是因为redis目前并没有太多的定时事件需要管理，redis以数据库服务器角色运行时，定时任务回调函数只有位于redis/src/redis.c下的serverCron函数，所有的定时任务都在这个函数下执行，也就是说，链表里面其实目前就一个节点元素，所以目前也无需实现高性能定时器。

Redis网络事件驱动模型代码：redis/src/目录下的ae.c, ae.h, ae_epoll.c, ae_evport.c, ae_select.c, ae_kqueue.c , ae_evport.c。其中ae.c/ae.h:头文件里定义了描述文件事件和事件时间的结构体， 即aeFileEvent和aeTimeEvent；事件驱动状态结构体aeEventLoop, 这个结构体只有一个名为eventloop的全局变量在整个服务器进程中；事件就绪回调函数指针aeFileProc和aeTimeProc；以及操作事件驱动模型的各种API（aeCreateEventLoop以及之后全部的函数声明）。ae_epoll.c, ae_select.c, ae_keque.c和ae_evport.c封装了select/epoll/kqueue等系统调用，Linux下当然不支持kqueue和evport。至于究竟选择哪一种I/O多路复用技术，在ae.c里有预处理控制，也就是说，这些源文件只有一个能最后被编译。优先选择epoll或者kqueue(FREEBSD和Mac OSX可用)，其次是select。

redis事件驱动整体流程：redis服务器main函数位于文件redis/src/redis.c, 事件驱动入口函数位于main函数的倒数第三行：

```text
aeMain(server.el); /* 实现代码位于ae.c */
```

这个函数调用aeProcessEvent进入事件循环，aeProcessEvent函数源码(同样位于ae.c源文件)：

```c
/* Process every pending time event, then every pending file event
 * (that may be registered by time event callbacks just processed).
 * Without special flags the function sleeps until some file event
 * fires, or when the next time event occurs (if any).
 *
 * If flags is 0, the function does nothing and returns.
 * if flags has AE_ALL_EVENTS set, all the kind of events are processed.
 * if flags has AE_FILE_EVENTS set, file events are processed.
 * if flags has AE_TIME_EVENTS set, time events are processed.
 * if flags has AE_DONT_WAIT set the function returns ASAP until all
 * the events that's possible to process without to wait are processed.
 *
 * The function returns the number of events processed. */
int aeProcessEvents(aeEventLoop *eventLoop, int flags)
{
    int processed = 0, numevents;

    /* Nothing to do? return ASAP */
    if (!(flags & AE_TIME_EVENTS) && !(flags & AE_FILE_EVENTS)) return 0;

    /* Note that we want call select() even if there are no
     * file events to process as long as we want to process time
     * events, in order to sleep until the next time event is ready
     * to fire. */
    if (eventLoop->maxfd != -1 ||
        ((flags & AE_TIME_EVENTS) && !(flags & AE_DONT_WAIT))) {
        int j;
        aeTimeEvent *shortest = NULL;
        struct timeval tv, *tvp;

        if (flags & AE_TIME_EVENTS && !(flags & AE_DONT_WAIT))
            shortest = aeSearchNearestTimer(eventLoop);
        if (shortest) {
            long now_sec, now_ms;

            /* Calculate the time missing for the nearest
             * timer to fire. */
            aeGetTime(&now_sec, &now_ms);
            tvp = &tv;
            tvp->tv_sec = shortest->when_sec - now_sec;
            if (shortest->when_ms < now_ms) {
                tvp->tv_usec = ((shortest->when_ms+1000) - now_ms)*1000;
                tvp->tv_sec --;
            } else {
                tvp->tv_usec = (shortest->when_ms - now_ms)*1000;
            }
            if (tvp->tv_sec < 0) tvp->tv_sec = 0;
            if (tvp->tv_usec < 0) tvp->tv_usec = 0;
        } else {
            /* If we have to check for events but need to return
             * ASAP because of AE_DONT_WAIT we need to set the timeout
             * to zero */
            if (flags & AE_DONT_WAIT) {
                tv.tv_sec = tv.tv_usec = 0;
                tvp = &tv;
            } else {
                /* Otherwise we can block */
                tvp = NULL; /* wait forever */
            }
        }

        numevents = aeApiPoll(eventLoop, tvp);
        for (j = 0; j < numevents; j++) {
            aeFileEvent *fe = &eventLoop->events[eventLoop->fired[j].fd];
            int mask = eventLoop->fired[j].mask;
            int fd = eventLoop->fired[j].fd;
            int rfired = 0;

	    /* note the fe->mask & mask & ... code: maybe an already processed
             * event removed an element that fired and we still didn't
             * processed, so we check if the event is still valid. */
            if (fe->mask & mask & AE_READABLE) {
                rfired = 1;
                fe->rfileProc(eventLoop,fd,fe->clientData,mask);
            }
            if (fe->mask & mask & AE_WRITABLE) {
                if (!rfired || fe->wfileProc != fe->rfileProc)
                    fe->wfileProc(eventLoop,fd,fe->clientData,mask);
            }
            processed++;
        }
    }
    /* Check time events */
    if (flags & AE_TIME_EVENTS)
        processed += processTimeEvents(eventLoop);

    return processed; /* return the number of processed file/time events */
}
```

读完后可以看出，aeProcess先根据全局变量eventloop中的距离当前最近时间事件来设置事件驱动器aeApiPoll函数(其实就是select, epoll_wait, kevent等函数的时间参数)的超时参数，aeApiPoll函数的实现位于每一个I/O多路复用器的封装代码中(即ae_epoll.c, ae_evport.c, ae_select.c, ae_kqueue.c , ae_evport.c)。aeApiPoll函数执行后，将就绪文件事件返回到eventloop的fired成员中，然后依次处理就绪的文件事件，执行其回调函数。最后，检查定时任务链表(processTimeEvents函数)， 执行时间任务。这就是redis服务器运行的大致主流程。