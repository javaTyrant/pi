# 类结构:

放锁拿锁.锁粒度细化.

```java
class LinkedBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, java.io.Serializable 
```

核心变量

```java
//容量
private final int capacity;
//Current number of elements,当前元素的大小
private final AtomicInteger count = new AtomicInteger();
//头
transient Node<E> head;
//尾
private transient Node<E> last;
//A variant of the "two lock queue" algorithm.
private final ReentrantLock takeLock = new ReentrantLock();
private final Condition notEmpty = takeLock.newCondition();
private final ReentrantLock putLock = new ReentrantLock();
private final Condition notFull = putLock.newCondition();
这几个变量一看就知道大概的流程了吧
```

先看put

```java
 public void put(E e) throws InterruptedException {
        if (e == null) throw new NullPointerException();
        // Note: convention in all put/take/etc is to preset local var
        // holding count negative to indicate failure unless set.
        int c = -1;
   			//构造一个新node节点
        Node<E> node = new Node<E>(e);
   			//获取put锁
        final ReentrantLock putLock = this.putLock;
   			//获取count
        final AtomicInteger count = this.count;
   			//加锁
        putLock.lockInterruptibly();
        try {
            /*
             * Note that count is used in wait guard even though it is
             * not protected by lock. This works because count can
             * only decrease at this point (all other puts are shut
             * out by lock), and we (or some other waiting put) are
             * signalled if it ever changes from capacity. Similarly
             * for all other uses of count in other wait guards.
             */
          	//如果放满了,等,等别人拿了,咱再放
            while (count.get() == capacity) {
                notFull.await();
            }
          	//被通知了,放入队列
            enqueue(node);
          	//自增,count等于-1,c才可能为0,所以要看take的count逻辑
          	//其他的线程是无法增加的,所以只可能被消费,如果消费成0了
          	//先get在自增,干
            c = count.getAndIncrement();
          	//未满通知,通知生产者
            if (c + 1 < capacity)
                notFull.signal();
        } finally {
          	//解锁
            putLock.unlock();
        }
   			//c什么时候等于0啊?为什么要通过这个c来判断呢
   			//count是多线程共享的
   			//没什么什么懵逼的事情是不能通过一个断点解决的
        if (c == 0)
          	//由于存在放锁和拿锁，这里可能拿锁一直在消费数据，count会变化。这里的if条件表示如果队列中还有1条数据
          	//在拿锁的条件对象notEmpty上唤醒正在等待的1个线程，表示队列里还有1条数据，可以进行消费
            signalNotEmpty();
 }

  /**
     * Signals a waiting take. Called only from put/offer (which do not
     * otherwise ordinarily lock takeLock.)
     */
    private void signalNotEmpty() {
        final ReentrantLock takeLock = this.takeLock;
        takeLock.lock();
        try {
            notEmpty.signal();
        } finally {
            takeLock.unlock();
        }
    }
    
```

再看take

```java
        E x;
        int c = -1;
        final AtomicInteger count = this.count;
        final ReentrantLock takeLock = this.takeLock;
        takeLock.lockInterruptibly();
        try {
          	//如果没有,等待
            while (count.get() == 0) {
                notEmpty.await();
            }
          	//在这里最小为1,并不会减小count的值
            x = dequeue();
          	//所以count的至少是1吧,0
            c = count.getAndDecrement();
            if (c > 1)
              	//
                notEmpty.signal();
        } finally {
            takeLock.unlock();
        }
				//
        if (c == capacity)
            signalNotFull();
        return x;
    }
```





```java
   /**
     * 简单的node
     */
    static class Node<E> {
        E item;

        /**
         * One of:
         * - the real successor Node
         * - this Node, meaning the successor is head.next
         * - null, meaning there is no successor (this is the last node)
         */
        Node<E> next;

        Node(E x) { item = x; }
    }

```

