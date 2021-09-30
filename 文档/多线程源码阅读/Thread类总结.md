## 1.线程的生命周期

### java.lang.Thread.State

NEW,

RUNNABLE,

BLOCKED,

WAITING,

TIMED_WAITING,

TERMINATED;

## 2.start和run的区别

start的时候会调用run的方法,由虚拟机调用

直接看源码

```java
public synchronized void start() {
    /**
     * This method is not invoked for the main method thread or "system"
     * group threads created/set up by the VM. Any new functionality added
     * to this method in the future may have to also be added to the VM.
     *
     * A zero status value corresponds to state "NEW".
     */
    if (threadStatus != 0)
        throw new IllegalThreadStateException();

    /* Notify the group that this thread is about to be started
     * so that it can be added to the group's list of threads
     * and the group's unstarted count can be decremented. */
    group.add(this);

    boolean started = false;
    try {
        start0();
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
            /* do nothing. If start0 threw a Throwable then
              it will be passed up the call stack */
        }
    }
}
```

```java
@Override
public void run() {
    if (target != null) {
        target.run();
    }
}
```

## 3.Thread.join的作用

Waits for this thread to die.

## 4.Thread.join的实现原理

看了源码,就一目了然了吧

```java
public final synchronized void join(long millis)
throws InterruptedException {
    long base = System.currentTimeMillis();
    long now = 0;

    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (millis == 0) {
      //只有isAlive就一直等
        while (isAlive()) {
          	//一看就是native方法,而且是Object的方法
          	// public final native void wait(long timeout) throws InterruptedException;
            wait(0);
        }
    } else {
        while (isAlive()) {
            long delay = millis - now;
            if (delay <= 0) {
                break;
            }
            wait(delay);
            now = System.currentTimeMillis() - base;
        }
    }
}
```

isAlive:猜测start0执行成功之后会改变线程isAlive的状态.

```java
/**
 * Tests if this thread is alive. A thread is alive if it has
 * been started and has not yet died.
 *
 * @return  <code>true</code> if this thread is alive;
 *          <code>false</code> otherwise.
 */
public final native boolean isAlive();
```

## 5.什么时候会使用Thread.join



## 6.Java线程池是如何保证核心线程不被销毁的

[参考](https://blog.csdn.net/smile_from_2015/article/details/105259789)

结论:

线程池当未调用 shutdown 方法时，是通过队列的 take 方法阻塞核心线程（Worker）的 run 方法从而保证核心线程不被销毁的。

## 7.Object中的方法-万物通用的方法

clone

equals

finalize

getClass

hashCode

notify

notifyAll

wait

toString

## 8.什么是守护线程

daemon



## Thread源码解读

#### 定义:A thread is a thread of execution in a program. 

#### 类结构:Thread implements Runnable 



### 核心变量

