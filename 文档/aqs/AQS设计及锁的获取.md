# AQS设计及锁的获取

类结构

```java
AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable 
```

有两个内部类:

Node:i

ConditionObject:

默认的非公平的锁:

```java
void lock();

public void lock() {
    sync.lock();
}

Sync  abstract void lock();

final void lock() {
//如果锁被其他线程持有了,acquire(1)
if (compareAndSetState(0, 1))
	setExclusiveOwnerThread(Thread.currentThread());
	else
	acquire(1);
}
//如果tryAcquire为true,说明获取成功了,然后selfInterrupt一下,为什么要自我打断呢
//否则acquireQueued  
public final void acquire(int arg) {
if (!tryAcquire(arg) &&
	acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
	selfInterrupt();
}
1.tryAcquire(arg) //非公平的锁
        final boolean nonfairTryAcquire(int acquires) {
  					//获取当前线程
            final Thread current = Thread.currentThread();
  					//获取state,因为用到了cas,所以这边无需加锁
            int c = getState();
  					//如果state没有锁持有
            if (c == 0) {
              	//cas比较
                if (compareAndSetState(0, acquires)) {
                  	//设置持有锁的线程
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
  					//锁重入
            else if (current == getExclusiveOwnerThread()) {
              	//严谨啊,溢出为负数
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
  					//
            return false;
        }
1.返回false,那么先执行addWaiter
      private Node addWaiter(Node mode) {
  			//当前线程,mode?Node.EXCLUSIVE,独占锁
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        Node pred = tail;
        if (pred != null) {
          	//新节点的prew指向tail
            node.prev = pred;
          	//cas设置尾结点
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
  			//等待队列现在是空的，没有线程在等待。
				//其他线程在当前线程入队的过程中率先完成了入队，导致尾节点的值已经改变了，CAS操作失败
        enq(node);
        return node;
    }
		//自旋插入,知道所有的节点都放入到尾节点
    private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { // Must initialize
                //头结点是空,dummynode
                if (compareAndSetHead(new Node()))
                  	//tail也是空node
                    tail = head;
            } else {
              	//新节点的前序指向tail
                node.prev = t;
              	//把node设置成tail
                if (compareAndSetTail(t, node)) {      
    								//此时node已经变成tail了
                    t.next = node;
                    return t;
                }
            }
        }
    }
//补充
如何将尾结点加入到sync queue呢
1.设置node的前驱节点为当前的尾结点:node.prev = t
2.修改tail,是他指向当前节点
3.修改原来的尾结点,让他的next指向当前节点
这里的三步并不是原子操作,第一步很容易成功,第二步cas操作,并发的条件下会失败,只有一个节点成为伟节点,其他的节点失败,回到循环中
	//请求入完队列之后
  acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
  (1) 能执行到该方法, 说明addWaiter 方法已经成功将包装了当前Thread的节点添加到了等待队列的队尾
(2) 该方法中将再次尝试去获取锁
(3) 在再次尝试获取锁失败后, 判断是否需要把当前线程挂起
  		//最难啃的骨头
      final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
  
  
  
  
  
  
  
  
  
  

```