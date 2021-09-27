## 任务执行

#### ExecutorService:

```java
<T> Future<T> submit(Callable<T> task);
```

#### AbstractExecutorService:模板类

```java
  public <T> Future<T> submit(Callable<T> task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<T> ftask = newTaskFor(task);
    		//线程池实现:runable
        execute(ftask);
        return ftask;
    }
```

封装成FutureTask.

```java
//FutureTask是runnable
protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
  	//-->RunnableFuture<V>->extends Runnable, Future<V>
    return new FutureTask<T>(callable);
}
```

FutureTask的构造器

```java
   public FutureTask(Callable<V> callable) {
        if (callable == null)
            throw new NullPointerException();
        this.callable = callable;
        this.state = NEW;       // ensure visibility of callable
    }
```

```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    /*
     * Proceed in 3 steps:
     *
     * 1. If fewer than corePoolSize threads are running, try to
     * start a new thread with the given command as its first
     * task.  The call to addWorker atomically checks runState and
     * workerCount, and so prevents false alarms that would add
     * threads when it shouldn't, by returning false.
     *
     * 2. If a task can be successfully queued, then we still need
     * to double-check whether we should have added a thread
     * (because existing ones died since last checking) or that
     * the pool shut down since entry into this method. So we
     * recheck state and if necessary roll back the enqueuing if
     * stopped, or start a new thread if there are none.
     *
     * 3. If we cannot queue task, then we try to add a new
     * thread.  If it fails, we know we are shut down or saturated
     * and so reject the task.
     */
    int c = ctl.get();
  	//工作线程小于核心线程
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
  	//如果isRunning且能入队列成功
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    else if (!addWorker(command, false))
        reject(command);
}
```

**线程池添加线程的逻辑**

```java
private boolean addWorker(Runnable firstTask, boolean core) {
  	//label
    retry:
  	//死循环,每次都要获取最新的ctl
    for (int c = ctl.get();;) {
        // Check if queue empty only if necessary.
        if (runStateAtLeast(c, SHUTDOWN) && (runStateAtLeast(c, STOP)
                || firstTask != null
                || workQueue.isEmpty())
           )
            return false;
				//死循环开始
        for (;;) {
          	//工作线程的数量不能大于最大线程的数量
            if (workerCountOf(c)// if true use corePoolSize as bound
                >= ((core ? corePoolSize : maximumPoolSize) & COUNT_MASK))
                return false;
          	//cas添加线程数量,如果成功,退出循环
            if (compareAndIncrementWorkerCount(c))
                break retry;
          	//重新加载一次
            c = ctl.get();  // Re-read ctl
          	//如果state状态大于等于shutdown,回到label处
            if (runStateAtLeast(c, SHUTDOWN))
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }
    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
      	//构造新worker,里面会new一个thread, this.thread = getThreadFactory().newThread(this);
        w = new Worker(firstTask);
      	//获取thread,然后用这个线程执行任务
        final Thread t = w.thread;
        //t不为空的话
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
          	//锁住
            mainLock.lock();
            try {
                // Recheck while holding lock.
                // Back out on ThreadFactory failure or if
                // shut down before lock acquired.
                int c = ctl.get();
                if (isRunning(c) || (runStateLessThan(c, STOP) && firstTask == null)) {
                  	//线程的生命周期
                    if (t.isAlive()) // precheck that t is startable
                        throw new IllegalThreadStateException();
                  	//添加到workers里
                    workers.add(w);
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {
                //触发任务
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        if (! workerStarted)
            addWorkerFailed(w);
    }
    return workerStarted;
}
```

new Worker(firstTask).thread.start()

Worker.start()**->**Worker.run()->runWorker.

```java
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
      	//死循环,没有task会阻塞
        while (task != null || (task = getTask()) != null) {
            w.lock();
            // If pool is stopping, ensure thread is interrupted;
            // if not, ensure thread is not interrupted.  This
            // requires a recheck in second case to deal with
            // shutdownNow race while clearing interrupt
            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                //打断线程
                wt.interrupt();
            try {
              	//钩子
                beforeExecute(wt, task);
                try {
                    task.run();
                  	//钩子
                    afterExecute(task, null);
                } catch (Throwable ex) {
                    afterExecute(task, ex);
                    throw ex;
                }
            } finally {
                task = null;
              	//完成任务++
                w.completedTasks++;
                //解锁
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        processWorkerExit(w, completedAbruptly);
    }
}
```

getTask如果没有任务是阻塞的.

```java
private Runnable getTask() {
    boolean timedOut = false; // Did the last poll() time out?

    for (;;) {
        int c = ctl.get();

        // Check if queue empty only if necessary.
        if (runStateAtLeast(c, SHUTDOWN)
            && (runStateAtLeast(c, STOP) || workQueue.isEmpty())) {
            decrementWorkerCount();
            return null;
        }

        int wc = workerCountOf(c);

        // Are workers subject to culling?
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

        if ((wc > maximumPoolSize || (timed && timedOut))
            && (wc > 1 || workQueue.isEmpty())) {
            if (compareAndDecrementWorkerCount(c))
                return null;
            continue;
        }

        try {
            Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                workQueue.take();//take是阻塞的,牛逼
            if (r != null)
                return r;
            timedOut = true;
        } catch (InterruptedException retry) {
            timedOut = false;
        }
    }
}
```

## 线程中断

问:线程池是如何清理线程的?非核心线程超时清理的流程.

```java
    final void runWorker(Worker w) {
        Thread wt = Thread.currentThread();
        Runnable task = w.firstTask;
        w.firstTask = null;
        w.unlock(); // allow interrupts
        boolean completedAbruptly = true;
        try {
            while (task != null || (task = getTask()) != null) {
               ...
            }
            completedAbruptly = false;
        } finally {
          	//什么时候会走到这步
            processWorkerExit(w, completedAbruptly);
        }
    }
```

