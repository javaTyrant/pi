本文介绍的 HashedWheelTimer 是来自于 Netty 的工具类，在 netty-common 包中。它用于实现延时任务。另外，下面介绍的内容和 Netty 无关。

如果你看过 Dubbo 的源码，一定会在很多地方看到它。在需要失败重试的场景中，它是一个非常方便好用的工具。

本文将会介绍 HashedWheelTimer 的使用，以及在后半部分分析它的源码实现。

## 接口概览

在介绍它的使用前，先了解一下它的接口定义，以及和它相关的类。

`HashedWheelTimer` 是接口 `io.netty.util.Timer` 的实现，从面向接口编程的角度，我们其实不需要关心 HashedWheelTimer，只需要关心接口类 Timer 就可以了。这个 Timer 接口只有两个方法：

```java
public interface Timer {
    // 创建一个定时任务
    Timeout newTimeout(TimerTask task, long delay, TimeUnit unit);
    // 停止所有的还没有被执行的定时任务
    Set<Timeout> stop();
}
```

```java
public interface TimerTask {
    void run(Timeout timeout) throws Exception;
}
```

```java
public interface Timeout {
    Timer timer();
    TimerTask task();
    boolean isExpired();
    boolean isCancelled();
    boolean cancel();
}
```

它持有上层的 Timer 实例，和下层的 TimerTask 实例，然后取消任务的操作也在这里面。

