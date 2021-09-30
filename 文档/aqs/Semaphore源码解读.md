概念: A counting semaphore.Conceptually, a semaphore maintains a set of permits. 

信号量维护了一组许可.

acquire阻塞直到有许可,然后获取

release增加一个

公平和非公平都是一样的,和锁一样,都是多了一个hasQueuedPredecessors

## 1.Acquire

### 1.1sync.acquireSharedInterruptibly(1);

acquireSharedInterruptibly是AQS的方法

```java
    public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
      	//AQS没有实现,需要自己实现,不同的工具类的逻辑是一样的,可以对比下闭锁和信号量的实现差异,也就是这两个工具类的差异
        if (tryAcquireShared(arg) < 0)
            doAcquireSharedInterruptibly(arg);
    }
```

#### 1.1.1tryAcquireShared

```java
   protected int tryAcquireShared(int acquires) {
     				//死循环等待
            for (;;) {
              	//是否有等待的节点
                if (hasQueuedPredecessors())
                    return -1;
              	//获取可用的
                int available = getState();
              	//可用的减去获取的
                int remaining = available - acquires;
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }
```



#### 1.1.2doAcquireSharedInterruptibly

