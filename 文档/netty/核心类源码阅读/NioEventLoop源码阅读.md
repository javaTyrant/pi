## NioEventLoop源码阅读

类结构

```java
NioEventLoop extends SingleThreadEventLoop 
   abstract class SingleThreadEventLoop extends SingleThreadEventExecutor implements EventLoop 
   abstract class SingleThreadEventExecutor extends AbstractScheduledEventExecutor implements 				    							OrderedEventExecutor  
   abstract class AbstractScheduledEventExecutor extends AbstractEventExecutor
   abstract class AbstractEventExecutor extends AbstractExecutorService implements EventExecutor	    
   interface EventLoop extends OrderedEventExecutor, EventLoopGroup  
   interface EventExecutor extends EventExecutorGroup
   interface EventExecutorGroup extends ScheduledExecutorService, Iterable<EventExecutor>
```

