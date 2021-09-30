# guava RateLimiter算法详解

```java
public void testSimple() {
        //一秒五个请求
        RateLimiter limiter = RateLimiter.create(5.0, stopwatch);
        limiter.acquire(); // R0.00, since it's the first request
        limiter.acquire(); // R0.20
        limiter.acquire(); // R0.20
        assertEvents("R0.00", "R0.20", "R0.20");
}
```

### 1.RateLimiter.create()

需要一个rate,一个秒表

```java
  static RateLimiter create(double permitsPerSecond, SleepingStopwatch stopwatch) {
        /* maxBurstSeconds */
        RateLimiter rateLimiter = new SmoothBursty(stopwatch, 1.0);
        rateLimiter.setRate(permitsPerSecond);
        return rateLimiter;
    }
```

SleepingStopwatch是RateLimiter的内部类

![image-20201012153650168](/Users/lumac/Library/Application Support/typora-user-images/image-20201012153650168.png)

**什么是Stopwatch**:An object that measures elapsed time in nanoseconds.测量消耗时间的对象.非线程安全的.

这个类比System#nanoTime的优势在:

- An alternate time source can be substituted, for testing or performance reasons.

