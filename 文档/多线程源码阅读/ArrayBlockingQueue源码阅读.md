我们已经看过LinkedBlockingQueue这个类的源码了,猜一下,Array这个类,最多就是数据结构的不同了吧.

Linked底层是个node,那么这个类底层肯定是Array

数据结构会怎么影响到这个类的设计,这更应该是我们应该关注的重点.

注意:这个类是有界的,而LinkedBlockingQueue是optionally-bounded.

## 核心变量

```java
/** The queued items */
final Object[] items;

/** items index for next take, poll, peek or remove */
int takeIndex;

/** items index for next put, offer, or add */
int putIndex;

/** Number of elements in the queue */
int count;

/*
 * Concurrency control uses the classic two-condition algorithm
 * found in any textbook.
 */

/** Main lock guarding all access */
final ReentrantLock lock;

/** Condition for waiting takes */
private final Condition notEmpty;

/** Condition for waiting puts */
private final Condition notFull;

/**
 * Shared state for currently active iterators, or null if there
 * are known not to be any.  Allows queue operations to update
 * iterator state.
 */
transient Itrs itrs = null;
```

## put方法

```java
      	public void put(E e) throws InterruptedException {
        //元素不允许为空  
        checkNotNull(e);
        //锁
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
          	//如果满了,等
            while (count == items.length)
                notFull.await();
          	//如数组
            enqueue(e);
        } finally {
            lock.unlock();
        }  
```

```java
//关注下putIndex的逻辑
private void enqueue(E x) {
    // assert lock.getHoldCount() == 1;
    // assert items[putIndex] == null;
    final Object[] items = this.items;
    items[putIndex] = x;
    if (++putIndex == items.length)
        putIndex = 0;
    count++;
    notEmpty.signal();
}
```

## take方法

```java
   public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == 0)
                notEmpty.await();
            return dequeue();
        } finally {
            lock.unlock();
        }
    }

```

```java
 /**
     * Extracts element at current take position, advances, and signals.
     * Call only when holding lock.
     */
    private E dequeue() {
        // assert lock.getHoldCount() == 1;
        // assert items[takeIndex] != null;
        final Object[] items = this.items;
        @SuppressWarnings("unchecked")
        E x = (E) items[takeIndex];
        items[takeIndex] = null;
        if (++takeIndex == items.length)
            takeIndex = 0;
        count--;
        if (itrs != null)
            itrs.elementDequeued();
        notFull.signal();
        return x;
    }

```

