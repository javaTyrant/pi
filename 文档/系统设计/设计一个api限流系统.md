## Designing an API Rate Limiter

### 限流规则



### 限流算法

#### 固定时间窗口限流算法

优点简单:

缺点:这种算法的限流策略过于粗略，无法应对 两个时间窗口临界时间内的突发流量。

#### 滑动时间窗口限流算法

```java
public class SlidingWindowRateLimiter implements RateLimiter, Runnable {
    /**
     * 阈值
     */
    private final Integer limitCount;

    /**
     * 当前通过的请求数
     */
    private final AtomicInteger passCount;

    /**
     * 窗口数
     */
    private final Integer windowSize;

    /**
     * 每个窗口时间间隔大小
     */
    private final long windowPeriod;

    /**
     * 时间单位
     */
    private final TimeUnit timeUnit;
    /**
     * windows数组.
     */
    private Window[] windows;

    /**
     * 窗口索引.
     */
    private volatile Integer windowIndex = 0;

    /**
     * 锁.
     */
    private final Lock lock = new ReentrantLock();

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(SlidingWindowRateLimiter.class);

    public SlidingWindowRateLimiter(Integer limitCount) {
        // 默认统计qps, 窗口大小5
        this(limitCount, 5, 200, TimeUnit.MILLISECONDS);
    }

    /**
     * 统计总时间 = windowSize * windowPeriod
     */
    public SlidingWindowRateLimiter(Integer limitCount, Integer windowSize, Integer windowPeriod, TimeUnit timeUnit) {
        this.limitCount = limitCount;
        this.windowSize = windowSize;
        this.windowPeriod = windowPeriod;
        this.timeUnit = timeUnit;
        this.passCount = new AtomicInteger(0);
        //
        this.initWindows(windowSize);
        //
        this.startResetTask();
    }

    @Override
    public boolean tryAcquire() throws InternalErrorException {

        lock.lock();
        if (passCount.get() > limitCount) {
            throw new InternalErrorException("限流啦.");
        }
        //
        windows[windowIndex].passCount.incrementAndGet();
        //
        passCount.incrementAndGet();

        lock.unlock();
        return true;
    }

    private void initWindows(Integer windowSize) {
        //构造一个window数组.
        windows = new Window[windowSize];
        for (int i = 0; i < windowSize; i++) {
            windows[i] = new Window();
        }
    }

    private void startResetTask() {
        //
        ScheduledExecutorService scheduledExecutorService = Executors.newSingleThreadScheduledExecutor();
        //固定的
        scheduledExecutorService.scheduleAtFixedRate(this, windowPeriod, windowPeriod, timeUnit);
    }

    //每隔200毫秒执行一次.
    @Override
    public void run() {
        //获取当前窗口索引.取模.因为窗口的数量是固定.最后一个会变成第0个
        Integer curIndex = (windowIndex + 1) % windowSize;
        log.info("infoResetTask, curIndex = {}", curIndex);
        //重置当前窗口索引通过数量，并获取上一次通过数量.Atomically sets to the given value and returns the old value.
        Integer count = windows[curIndex].passCount.getAndSet(0);
        //
        windowIndex = curIndex;
        //总通过数量 减去 当前窗口上次通过数量
        passCount.addAndGet(-count);
        log.info("infoResetTask, curOldCount = {}, passCount = {}, windows = {}", count, passCount, windows);
    }

    @Data
    static class Window {
        /**
         * 一个窗口内通过的数量.
         */
        private AtomicInteger passCount;

        public Window() {
            this.passCount = new AtomicInteger(0);
        }
    }
}
```

#### 令牌桶限流算法

​    令牌桶算法是[网络流量](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%E7%BD%91%E7%BB%9C%E6%B5%81%E9%87%8F)整形（Traffic Shaping）和速率限制（Rate Limiting）中最常使用的一种算法。典型情况下，令牌桶算法用来控制发送到网络上的数据的数目，并允许突发数据的发送。

  大小固定的令牌桶可自行以恒定的速率源源不断地产生令牌。如果令牌不被消耗，或者被消耗的速度小于产生的速度，令牌就会不断地增多，直到把桶填满。后面再产生的令牌就会从桶中溢出。最后桶中可以保存的最大令牌数永远不会超过桶的大小。                  

guava的RateLimiter

#### 漏桶限流算法



### 限流模式



### 集成使用

