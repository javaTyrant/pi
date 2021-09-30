## netty内存泄露指的是什么

![image-20210208223730978](/Users/lumac/Library/Application Support/typora-user-images/image-20210208223730978.png)

## netty内存泄露检测核心思路

引用计数+弱引用

## netty内存泄露检测的源码解析

![image-20210208225308226](/Users/lumac/Library/Application Support/typora-user-images/image-20210208225308226.png)

![image-20210208225609722](/Users/lumac/Library/Application Support/typora-user-images/image-20210208225609722.png)

## 堆外内存释放的底层实现

Netty的堆外内存是基于java的DirectByteBuffer对象基础上实现的.如何释放呢?

java.nio提供的DirectByteBuffer提供了sun.misc.Cleaner类的clean()方法，进行系统调用释放堆外内存，触发clean()方法的情况有2种

1.应用层序主动调用

```
ByteBuffer buf = ByteBuffer.allocateDirect(1);
((DirectBuffer) byteBuffer).cleaner().clean();
```

2.gc回收

