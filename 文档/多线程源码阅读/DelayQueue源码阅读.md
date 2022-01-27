# DelayQueue源码阅读

**思考:有更优秀的替代版本吗?**

队列的元素:

用于存放拥有过期时间的元素的阻塞队列.元素的过期时间如何定义?

DelayQueue内部用PriorityQueue实现，可认为DelayQueue=BlockingQueue+PriorityQueue+Delayed

类结构:

```java
class DelayQueue<E extends Delayed> extends AbstractQueue<E>
    implements BlockingQueue<E>
```

## 父接口

BlockingQueue

```java
interface  Queue<E>
	boolean add(E e); //
	boolean offer(E e);//
	E remove();//
	E poll();//
	E element();//
	E peek();//
interface BlockingQueue<E> extends Queue<E>

```



## 类介绍

An unbounded [blocking queue](../../../java/util/concurrent/BlockingQueue.html) of `Delayed` elements, in which an element can only be taken when its delay has expired.The *head* of the queue is that`Delayed` element whose delay expired furthest in the past. If no delay has expired there is no head and `poll` will return `null`. Expiration occurs when an element's `getDelay(TimeUnit.NANOSECONDS)` method returns a value less than or equal to zero. Even though unexpired elements cannot be removed using `take` or `poll`, they are otherwise treated as normal elements. For example, the `size` method returns the count of both expired and unexpired elements. This queue does not permit null elements.

This class and its iterator implement all of the *optional* methods of the [`Collection`](../../../java/util/Collection.html) and [`Iterator`](../../../java/util/Iterator.html) interfaces. The Iterator provided in method [`iterator()`](../../../java/util/concurrent/DelayQueue.html#iterator--)is *not* guaranteed to traverse the elements of the DelayQueue in any particular order.

1.什么时候**过期**:当这个元素的getDelay返回的值小于等于0

2.对应允许空元素吗?no

3.**未过期**的元素可以take或者poll吗?no

4.head of the queue:无过期元素为null,有已过期的元素才有head

5.元素的迭代保证顺序吗?no

6.什么是`Delayed` elements:读了几个单词我们就知道要看看什么是`Delayed` elements.后面就解释了,这个元素只有在他的延迟到期后,才能取出来.队列的头是第一个已经过期的延迟元素.如果没有元素过期,那么就不会有head,且poll会返回null.

## `Delayed`

A mix-in style interface for marking objects that should be acted upon after a given delay.

An implementation of this interface must define a `compareTo` method that provides an ordering consistent with its `getDelay` method.

```
Delayed extends Comparable<Delayed> 
```

所以有两个方法要实现:一个getDelay一个是compareTo

## 核心变量

```java
//
private final PriorityQueue<E> q = new PriorityQueue<E>();
//
private Thread leader = null;
//
private final Condition available = lock.newCondition();
```

## 核心方法

**poll和take的区别**

还有个带timeout的poll.

```java
public E poll() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        //什么时候put呢?
        E first = q.peek();
        //为空或者没有过期
        if (first == null || first.getDelay(NANOSECONDS) > 0)
            return null;
        else
            //直接poll
            return q.poll();
    } finally {
        lock.unlock();
    }
}
```

**take的逻辑**

```java
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        //死循环
        for (;;) {
            E first = q.peek();
            //如果为空,就等.等说明了什么呢?说明队列没有元素,等生成者生产.
            if (first == null)
                available.await();
            else {
                //否则获取delay
                long delay = first.getDelay(NANOSECONDS);
                //过期时间到了
                if (delay <= 0)
                    return q.poll();
                //
                first = null; // don't retain ref while waiting
                //leader不为空等待
                if (leader != null)
                    available.await();
                else {//leader为空
                    Thread thisThread = Thread.currentThread();
                    leader = thisThread;
                    try {
                        //等待delay的时间.
                        available.awaitNanos(delay);
                    } finally {
                        //如果leader是当前的线程,leader置空.
                        if (leader == thisThread)
                            leader = null;
                    }
                }
            }
        }
    } finally {
        //什么时候通知.leader为空了,且队列头有值.
        if (leader == null && q.peek() != null)
            //通知.谁呢?
            available.signal();
        lock.unlock();
    }
}
```



```java
public boolean offer(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        //放入队列
        q.offer(e);
        //如果相等?什么时候会不等呢?
        //如果放入的就是最快过期的
        if (q.peek() == e) {
            //
            leader = null;
            //t通知.
            available.signal();
        }
        return true;
    } finally {
        lock.unlock();
    }
}
```



```java

```

## 扩展:Follower Leader 模式

https://www.cnblogs.com/yeyang/p/12580600.html

简单粗暴的per client per thread,后果很严重.线程会暴涨.

#### 半同步-半异步模式:

**工作原理**: java 1.4给我们带来了NIO编程模型,由于它的读写操作都是无阻塞的,这样使我们能够只用一个线程处理所有的IO事件,当然我们不会真的只用一个线程来处理,比较常见的编写NIO网络服务程序的模型是半同步-半异步模式，其实现原理大体上是单线程同步处理网络IO请求,当有请求到达时,将该请求放入一个工作队列中,由另外的线程处理,主线程继续等待新的网络IO请求。

**缺点:**1.使用工作队列带来的内存的动态分配问题。
2.每次网络IO请求,总是分配给另外一个线程处理,这样频繁的线程的context switching也会影响性能

#### 领导者/追随者（Leader/Follower）模式

针对以上缺点，解决的方案是使用Leader-Follower线程模型,它的基本思想是所有的线程被分配成两种角色: 

Leader和Follower,一般同时只有一个Leader线程,所有的Follower线程排队等待成为Leader线程,线程池启动时自动产生一个Leader负责等待网络IO事件,当有一个事件产生时,Leader线程首先通知一个Follower线程,并将其提拔为新的Leader,然后自己去处理这个网络事件,处理完毕后加入Follower线程等待队列,等待重新成为Leader. 

关键点：
（1）只有1个leader线程，可以有若干的follower线程；
（2）线程有3种状态：leading/processing/following；
（3）有一把锁，抢到的就是leading；
（4）事件来到时，leading线程会对其进行处理，从而转化为processing状态；
（5）处理完成后，尝试抢锁，抢到则又变为leading，否则变为followering；
（6）followering不干事，就是抢锁，力图成为leading；

这种线程模型的优点：

1）这个线程模型主要解决了内存的动态分配问题,我们不需要不断的将到来的网络IO事件放入队列,

2）并且等待网络IO事件的线程在等到网络事件的发生后,是自己处理的这个事件,也就是说没有context switching的过程.

Leader-Follower模式

计算机版本：

1. 有若干个线程(一般组成线程池)用来处理大量的事件
2. 有一个线程作为领导者，等待事件的发生；其他的线程作为追随者，仅仅是睡眠。
3. 假如有事件需要处理，领导者会从追随者中指定一个新的领导者，自己去处理事件。
4. 唤醒的追随者作为新的领导者等待事件的发生。
5. 处理事件的线程处理完毕以后，就会成为追随者的一员，直到被唤醒成为领导者。
6. 假如需要处理的事件太多，而线程数量不够(能够动态创建线程处理另当别论)，则有的事件可能会得不到处理。

简单理解，就是最多只有一个线程在处理，其他线程在睡眠。

### DelayQueue中的follower leader模式:

因为我们知道这是一个多线程安全的队列,肯定会有这种情况,多个线程都会同时的take,那么先来的线程会变成leader,等待取,取完了,下一个线程才可能取,后来的线程必须等待之前释放leader.才能获取到take.