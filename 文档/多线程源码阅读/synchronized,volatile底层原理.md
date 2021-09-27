# 使用场景

synchronized有两种形式上锁，一个是对方法上锁，一个是构造同步代码块。他们的底层实现其实都一样，在进入同步代码之前先获取锁，获取到锁之后锁的计数器+1，同步代码执行完锁的计数器-1，如果获取失败就阻塞式等待锁的释放。只是他们在同步块识别方式上有所不一样，从class字节码文件可以表现出来，一个是通过方法flags标志，一个是monitorenter和monitorexit指令操作。

1.修饰普通同步方法

可以看到在add方法的flags里面多了一个`ACC_SYNCHRONIZED`标志，这标志用来告诉JVM这是一个同步方法，在进入该方法之前先获取相应的锁，锁的计数器加1，方法结束后计数器-1，如果获取失败就阻塞住，知道该锁被释放。

2.修饰静态同步方法

对class加锁,字节码和上面相同

3.修饰同步代码块

从反编译的同步代码块可以看到同步块是由monitorenter指令进入，然后monitorexit释放锁，在执行monitorenter之前需要尝试获取锁，如果这个对象没有被锁定，或者当前线程已经拥有了这个对象的锁，那么就把锁的计数器加1。当执行monitorexit指令时，锁的计数器也会减1。当获取锁失败时会被阻塞，一直等待锁被释放。

但是为什么会有两个monitorexit呢？其实第二个monitorexit是来处理异常的，仔细看反编译的字节码，正常情况下第一个monitorexit之后会执行`goto`指令，而该指令转向的就是23行的`return`，也就是说正常情况下只会执行第一个monitorexit释放锁，然后返回。而如果在执行中发生了异常，第二个monitorexit就起作用了，它是由编译器自动生成的，在发生异常时处理异常然后释放掉锁。

# volatile底层实现原理

定义:Java编程语言允许线程访问共享变量，为了确保共享变量能够被准确和一致性地更新，线程应该确保通过排它锁单独获得这个变量。

通俗来说就是**一个字段被volatile修饰，Java的内存模型确保所有的线程看到的这个变量值是一致的，**但是它并不能保证多线程的原子操作。这就是所谓的线程可见性。**我们要知道他是不能保证原子性的**。

![image-20210208091216102](/Users/lumac/Library/Application Support/typora-user-images/image-20210208091216102.png)

举个例子：

```
i++;
```

当线程运行这行代码时，首先会从主内存中读取i，然后复制一份到CPU高速缓存中,接着CPU执行+1的操作，再将+1后的数据写在缓存中，最后一步才是刷新到主内存中。在单线程时没有问题，多线程就有问题了。

如下：假如有两个线程A、B都执行这个操作（i++），按照我们正常的逻辑思维主存中的i值应该=3，但事实是这样么？

> 分析如下：
>
> 两个线程从主存中读取i的值（1）到各自的高速缓存中，然后线程A执行+1操作并将结果写入高速缓存中，最后写入主存中，此时主存i==2,线程B做同样的操作，主存中的i仍然=2。所以最终结果为2并不是3。这种现象就是缓存一致性问题。

解决缓存一致性方案有两种：

1. **通过在总线加LOCK#锁的方式；**
2. **通过缓存一致性协议。**

但是方案1存在一个问题，**它是采用一种独占的方式来实现的，即总线加LOCK#锁的话，只能有一个CPU能够运行，其他CPU都得阻塞，效率较为低下。**

第二种方案，**缓存一致性协议（MESI协议）它确保每个缓存中使用的共享变量的副本是一致的。**所以JMM就解决这个问题。

有volatile修饰的共享变量进行写操作的时候会多出Lock前缀的指令，该指令在多核处理器下会引发两件事情。

1. 将当前处理器缓存行数据刷写到系统主内存。
2. 这个刷写回主内存的操作会使其他CPU缓存的该共享变量内存地址的数据无效。
3. volatile通过内存屏障实现了防止指令重排的目的。

Java内存屏障主要有Load和Store两类：

- 对Load Barrier来说，在读指令前插入读屏障，可以让高速缓存中的数据失效，重新从主内存加载数据
- 对Store Barrier来说，在写指令之后插入写屏障，能让写入缓存的最新数据写回到主内存
  为了实现volatile的内存语义，编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。然而，对于编译器来说，发现一个最优布置来最小化插入屏障的总数几乎不可能，为此，Java内存模型采取保守策略：
- 在每个volatile写操作的前面插入一个StoreStore屏障。
- 在每个volatile写操作的后面插入一个StoreLoad屏障。
- 在每个volatile读操作的后面插入一个LoadLoad屏障。
- 在每个volatile读操作的后面插入一个LoadStore屏障。

## 缓存一致性协议（MESI协议)



```java
我将一部分常量池和方法的一些代码删掉了，我们只看这几个测试方法

  public static synchronized void test1();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_STATIC, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=0, args_size=0
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String static synchrozied
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 6: 0
        line 7: 8

  public synchronized void test2();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #5                  // String nostatic method syschronized
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 9: 0
        line 10: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  this   Lcom/nettylearn/Service/binarycode/syschoriznizedbytecodetest;

  public void test3();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=1
         0: aload_0
         1: getfield      #6                  // Field a:Ljava/lang/Object;
         4: dup
         5: astore_1
         6: monitorenter
         7: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
        10: ldc           #7                  // String code block test
        12: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        15: aload_1
        16: monitorexit
        17: goto          25
        20: astore_2
        21: aload_1
        22: monitorexit
        23: aload_2
        24: athrow
        25: return
      Exception table:
         from    to  target type
             7    17    20   any
            20    23    20   any
      LineNumberTable:
        line 12: 0
        line 13: 7
        line 14: 15
        line 15: 25
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      26     0  this   Lcom/nettylearn/Service/binarycode/syschoriznizedbytecodetest;
      StackMapTable: number_of_entries = 2
        frame_type = 255 /* full_frame */
          offset_delta = 20
          locals = [ class com/nettylearn/Service/binarycode/syschoriznizedbytecodetest, class java/lang/Object ]
          stack = [ class java/lang/Throwable ]
        frame_type = 250 /* chop */
          offset_delta = 4
}
SourceFile: "syschoriznizedbytecodetest.java"
```

