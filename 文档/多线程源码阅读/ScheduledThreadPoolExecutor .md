## 使用

```java
  public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }
		//ThreadPoolExecutor,构造了一个DelayedWorkQueue
    public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE,
              DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
              new DelayedWorkQueue());
    }
```

Additionally, it is almost never a good idea to set corePoolSize to zero or use allowCoreThreadTimeOut because this may leave the pool without threads to handle tasks once they become eligible to run.

## 类结构

```java
class ScheduledThreadPoolExecutor
        extends ThreadPoolExecutor
        implements ScheduledExecutorService 
```

实现原理



## ScheduledFuture

```java
public interface ScheduledFuture<V> extends Delayed, Future<V> 
```



## ScheduledFutureTask

类结构:

```java
   private class ScheduledFutureTask<V>
            extends FutureTask<V> implements RunnableScheduledFuture<V> {
}
```



## DelayedWorkQueue

1. DelayedWorkQueue的数据结构是怎样的
2. DelayedWorkQueue如何进行入队出队
3. DelayedWorkQueue如何实现延迟取出队

自己想想最适合的数据结构是什么呢.

类注释:1.heap-based,2.except that every ScheduledFutureTask also records its index into the heap array.

```sql
  /*
         * A DelayedWorkQueue is based on a heap-based data structure
         * like those in DelayQueue and PriorityQueue, except that
         * every ScheduledFutureTask also records its index into the
         * heap array. This eliminates the need to find a task upon
         * cancellation, greatly speeding up removal (down from O(n)
         * to O(log n)), and reducing garbage retention that would
         * otherwise occur by waiting for the element to rise to top
         * before clearing. But because the queue may also hold
         * RunnableScheduledFutures that are not ScheduledFutureTasks,
         * we are not guaranteed to have such indices available, in
         * which case we fall back to linear search. (We expect that
         * most tasks will not be decorated, and that the faster cases
         * will be much more common.)
         *
         * All heap operations must record index changes -- mainly
         * within siftUp and siftDown. Upon removal, a task's
         * heapIndex is set to -1. Note that ScheduledFutureTasks can
         * appear at most once in the queue (this need not be true for
         * other kinds of tasks or work queues), so are uniquely
         * identified by heapIndex.
         */

```





```java
 public ScheduledFuture<?> schedule(Runnable command,
                                       long delay,
                                       TimeUnit unit) {
        if (command == null || unit == null)
            throw new NullPointerException();
   			//封装成一个task
        RunnableScheduledFuture<Void> t = decorateTask(command,
            new ScheduledFutureTask<Void>(command, null,
                                          triggerTime(delay, unit),
                                          sequencer.getAndIncrement()));
        delayedExecute(t);
        return t;
  }
```

```java
 private void delayedExecute(RunnableScheduledFuture<?> task) {
        if (isShutdown())
            reject(task);
        else {
            super.getQueue().add(task);
            if (!canRunInCurrentRunState(task) && remove(task))
                task.cancel(false);
            else
                ensurePrestart();
        }
    }
```

```java
//Returns the nanoTime-based trigger time of a delayed action.
 private long triggerTime(long delay, TimeUnit unit) {
        return triggerTime(unit.toNanos((delay < 0) ? 0 : delay));
 }
```

```java
//如果为true就执行ensurePrestart();  
boolean canRunInCurrentRunState(RunnableScheduledFuture<?> task) {
				//没有shutdown返回true
        if (!isShutdown())
            return true;
  			//stop判断
        if (isStopped())
            return false;
  			//是否是周期?:
        return task.isPeriodic()
            ? continueExistingPeriodicTasksAfterShutdown
            : (executeExistingDelayedTasksAfterShutdown
               || task.getDelay(NANOSECONDS) <= 0);
    }

```



## 小思考

