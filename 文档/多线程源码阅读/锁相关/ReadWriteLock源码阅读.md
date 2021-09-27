```java
public interface ReadWriteLock {
    /**
     * Returns the lock used for reading.
     *
     * @return the lock used for reading
     */
    Lock readLock();

    /**
     * Returns the lock used for writing.
     *
     * @return the lock used for writing
     */
    Lock writeLock();
}
```

## 1.ReentrantReadWriteLock

五个内部类:
**Sync**

**FairSync**

**NonFairSync**

**ReadLock**
**WriteLock**

`ReentrantReadWriteLock`的核心是由一个基于AQS的同步器`Sync`构成，然后由其扩展出`ReadLock`（共享锁），`WriteLock`（排它锁）所组成。

读读是怎么做到无锁互斥的

1.读锁获取不到,进入等待队列

2.如果获取到读锁

读锁的获取主要实现是AQS中的`acquireShared`方法，其调用过程如下代码。

```java
// ReadLock
public void lock() {
    sync.acquireShared(1);
}
// AQS
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```

其中`doAcquireShared(arg)`方法是获取失败之后AQS中入队操作，等待被唤醒后重新获取，那么关键点就是`tryAcquireShared(arg)`方法，方法有点长，因此先总结出获取读锁所经历的步骤，获取的第一部分步骤如下：

- 操作1：读写需要互斥，因此当存在写锁并且持有写锁的线程不是该线程时获取失败。
- 操作2：是否存在等待写锁的线程，存在的话则获取读锁需要等待，避免写锁饥饿。(写锁优先级是比较高的)
- 操作3：CAS获取读锁，实际上是state字段的高16位自增。
- 操作4：获取成功后再ThreadLocal中记录当前线程获取读锁的次数。

**清单4：读锁获取的第一部分**

```java
protected final int tryAcquireShared(int unused) {
          Thread current = Thread.currentThread();
          int c = getState();
          // 操作1：存在写锁，并且写锁不是当前线程则直接去排队
          if (exclusiveCount(c) != 0 && //Returns the number of exclusive holds represented in count
              getExclusiveOwnerThread() != current)
              return -1;
					//目前不存在写锁
          int r = sharedCount(c);
          // 操作2：读锁是否该阻塞，对于非公平模式下写锁获取优先级会高，如果存在要获取写锁的线程则读锁需要让步，公平模式下则先来先到
          if (!readerShouldBlock() && 
              // 读锁使用高16位，因此存在获取上限为2^16-1
              r < MAX_COUNT &&
              // 操作3：CAS修改读锁状态，实际上是读锁状态+1
              compareAndSetState(c, c + SHARED_UNIT)) {
              // 操作4：执行到这里说明读锁已经获取成功，因此需要记录线程状态。
              if (r == 0) {
                  firstReader = current; // firstReader是把读锁状态从0变成1的那个线程
                  firstReaderHoldCount = 1;
              } else if (firstReader == current) { 
                  firstReaderHoldCount++;
              } else {
                  // 这些代码实际上是从ThreadLocal中获取当前线程重入读锁的次数，然后自增下。
                  HoldCounter rh = cachedHoldCounter; // cachedHoldCounter是上一个获取锁成功的线程
                  if (rh == null || rh.tid != getThreadId(current))
                      cachedHoldCounter = rh = readHolds.get();
                  else if (rh.count == 0)
                      readHolds.set(rh);
                  rh.count++;
              }
              return 1;
          }
          // 当操作2，操作3失败时执行该逻辑
          return fullTryAcquireShared(current);
      }
```

当操作2，操作3失败时会执行`fullTryAcquireShared(current)`，为什么会这样写呢？个人认为是一种补偿操作，**操作2与操作3失败并不代表当前线程没有读锁的资格**，并且这里的读锁是共享锁，有资格就应该被获取成功，因此给予补偿获取读锁的操作。在`fullTryAcquireShared(current)`中是一个循环获取读锁的过程，大致步骤如下：

- 操作5：等同于操作2，存在写锁，且写锁线程并非当前线程则直接返回失败
- 操作6：当前线程是重入读锁，这里只会偏向第一个获取读锁的线程以及最后一个获取读锁的线程，其他都需要去AQS中排队。
- 操作7：CAS改变读锁状态
- 操作8：同操作4，获取成功后再ThreadLocal中记录当前线程获取读锁的次数。

```java
final int fullTryAcquireShared(Thread current) {
           HoldCounter rh = null;
           // 最外层嵌套循环
           for (;;) {
               int c = getState();
               // 操作5：存在写锁，且写锁并非当前线程则直接返回失败
               if (exclusiveCount(c) != 0) {
                   if (getExclusiveOwnerThread() != current)
                       return -1;
                   // else we hold the exclusive lock; blocking here
                   // would cause deadlock.
               // 操作6：如果当前线程是重入读锁则放行
               } else if (readerShouldBlock()) {
                   // Make sure we're not acquiring read lock reentrantly
                   // 当前是firstReader，则直接放行,说明是已获取的线程重入读锁
                   if (firstReader == current) {
                       // assert firstReaderHoldCount > 0;
                   } else {
                       // 执行到这里说明是其他线程，如果是cachedHoldCounter（其count不为0）也就是上一个获取锁的线程则可以重入，否则进入AQS中排队
                       // **这里也是对写锁的让步**，如果队列中头结点为写锁，那么当前获取读锁的线程要进入队列中排队
                       if (rh == null) {
                           rh = cachedHoldCounter;
                           if (rh == null || rh.tid != getThreadId(current)) {
                               rh = readHolds.get();
                               if (rh.count == 0)
                                   readHolds.remove();
                           }
                       }
                       // 说明是上述刚初始化的rh，所以直接去AQS中排队
                       if (rh.count == 0)
                           return -1;
                   }
               }
               if (sharedCount(c) == MAX_COUNT)
                   throw new Error("Maximum lock count exceeded");
               // 操作7：修改读锁状态，实际上读锁自增操作
               if (compareAndSetState(c, c + SHARED_UNIT)) {
                   // 操作8：对ThreadLocal中维护的获取锁次数进行更新。
                   if (sharedCount(c) == 0) {
                       firstReader = current;
                       firstReaderHoldCount = 1;
                   } else if (firstReader == current) {
                       firstReaderHoldCount++;
                   } else {
                       if (rh == null)
                           rh = cachedHoldCounter;
                       if (rh == null || rh.tid != getThreadId(current))
                           rh = readHolds.get();
                       else if (rh.count == 0)
                           readHolds.set(rh);
                       rh.count++;
                       cachedHoldCounter = rh; // cache for release
                   }
                   return 1;
               }
           }
       }
```

## 写锁的获取逻辑

```java
   protected final boolean tryAcquire(int acquires) {
            /*
             * Walkthrough:
             * 1. If read count nonzero or write count nonzero
             *    and owner is a different thread, fail.
             * 2. If count would saturate, fail. (This can only
             *    happen if count is already nonzero.)
             * 3. Otherwise, this thread is eligible for lock if
             *    it is either a reentrant acquire or
             *    queue policy allows it. If so, update state
             *    and set owner.
             */
            Thread current = Thread.currentThread();
     				//第一次获取写锁的时候,state是0
            int c = getState();
						//独占的数量
            int w = exclusiveCount(c);
     				//所以第一次就不会执行这个逻辑直接走到下面
            if (c != 0) {
                // (Note: if c != 0 and w == 0 then shared count != 0)
              	//
                if (w == 0 || current != getExclusiveOwnerThread())
                    return false;
                if (w + exclusiveCount(acquires) > MAX_COUNT)
                    throw new Error("Maximum lock count exceeded");
                // Reentrant acquire
                setState(c + acquires);
                return true;
            }
     				//此时writer也不会阻塞,如果cas设置成功了,设置专属线程,返回true
            if (writerShouldBlock() ||
                !compareAndSetState(c, c + acquires))
                return false;
            setExclusiveOwnerThread(current);
            return true;
        }
```

## StampedLock