# AQS锁的释放

https://blog.csdn.net/foxException/article/details/108917338

https://segmentfault.com/a/1190000015752512

## 1.ReentrantLock

```java
public void unlock() {
    sync.release(1);
}
```

## 1.1AQS.release

AQS.release

```java
    public final boolean release(int arg) {
      	//如果释放成功
        if (tryRelease(arg)) {
          	//获取头结点指针
            Node h = head;
          	//
            if (h != null && h.waitStatus != 0)
              	//唤醒
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
```

### 1.1.1tryRelease

java.util.concurrent.locks.ReentrantLock.Sync#tryRelease

```java
protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
```

### 1.1.1.1unparkSuccessor

java.util.concurrent.locks.AbstractQueuedSynchronizer#unparkSuccessor

```java
private void unparkSuccessor(Node node) {
    /*
     * If status is negative (i.e., possibly needing signal) try
     * to clear in anticipation of signalling.  It is OK if this
     * fails or if status is changed by waiting thread.
     */
    int ws = node.waitStatus;
  	//如果waitstatus<0,更新成0.why
    if (ws < 0) compareAndSetWaitStatus(node, ws, 0);
    /*
     * Thread to unpark is held in successor, which is normally
     * just the next node.  But if cancelled or apparently null,
     * traverse backwards from tail to find the actual
     * non-cancelled successor.
     */
  	//获取当前节点的下一个节点
    Node s = node.next;
  	//如果不存在或者s的waitStatus是取消的状态
    if (s == null || s.waitStatus > 0) {
      	//s的下一个节点是取消的状态,置空
        s = null;
      	//从尾部开始遍历,一直遍历到t==null或者t == node
      	//为什么从尾巴开始呢?enq里有答案
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
              	//找到最靠近head的节点
                s = t;
    }
  	//开始执行unpark
    if (s != null)
        LockSupport.unpark(s.thread);
}
```

