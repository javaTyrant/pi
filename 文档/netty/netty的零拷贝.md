## 传统意义的零拷贝

Netty中的零拷贝与我们传统理解的零拷贝不太一样。
传统的零拷贝指的是数据传输过程中，不需要CPU进行数据的拷贝。主要是数据在用户空间与内核中间之间的拷贝。

```c
1.  `File.read(bytes)`
2.  `Socket.send(bytes)`
```

这种方式需要四次数据拷贝和四次上下文切换：

1. 数据从磁盘读取到内核的read buffer
2. 数据从内核缓冲区拷贝到用户缓冲区
3. 数据从用户缓冲区拷贝到内核的socket buffer
4. 数据从内核的socket buffer拷贝到网卡接口的缓冲区

明显上面的第二步和第三步是没有必要的，通过java的FileChannel.transferTo方法，可以避免上面两次多余的拷贝（当然这需要底层操作系统支持）

1. 调用transferTo,数据从文件由DMA引擎拷贝到内核read buffer
2. 接着DMA从内核read buffer将数据拷贝到网卡接口buffer

上面的两次操作都不需要CPU参与，所以就达到了零拷贝。

## Netty中的零拷贝

Netty中也用到了FileChannel.transferTo方法，所以Netty的零拷贝也包括上面将的操作系统级别的零拷贝。除此之外，在ByteBuf的实现上，Netty也提供了零拷贝的一些实现。

关于ByteBuffer，Netty提供了两个接口:

1. ByteBuf

2. ByteBufHolder

对于ByteBuf，Netty提供了多种实现：

1.Heap ByteBuf:直接在堆内存分配

2.Direct ByteBuf：直接在内存区域分配而不是堆内存

3.CompositeByteBuf：组合Buffer

### Direct Buffers

直接在内存区域分配空间，而不是在堆内存中分配。如果使用传统的堆内存分配，当我们需要将数据通过socket发送的时候，就需要从堆内存拷贝到直接内存，然后再由直接内存拷贝到网卡接口层。
 Netty提供的直接Buffer，直接将数据分配到内存空间，从而避免了数据的拷贝，实现了零拷贝。

### Composite Buffers

传统的ByteBuffer，如果需要将两个ByteBuffer中的数据组合到一起，我们需要首先创建一个size=size1+size2大小的新的数组，然后将两个数组中的数据拷贝到新的数组中。但是使用Netty提供的组合ByteBuf，就可以避免这样的操作，因为CompositeByteBuf并没有真正将多个Buffer组合起来，而是保存了它们的引用，从而避免了数据的拷贝，实现了零拷贝。

### 对于FileChannel.transferTo的使用

Netty中使用了FileChannel的transferTo方法，该方法依赖于操作系统实现零拷贝。

## 总结

Netty的零拷贝体现在三个方面：

> 1. Netty的接收和发送ByteBuffer采用DIRECT BUFFERS，使用堆外直接内存进行Socket读写，不需要进行字节缓冲区的二次拷贝。如果使用传统的堆内存（HEAP BUFFERS）进行Socket读写，JVM会将堆内存Buffer拷贝一份到直接内存中，然后才写入Socket中。相比于堆外直接内存，消息在发送过程中多了一次缓冲区的内存拷贝。

> 2. Netty提供了组合Buffer对象，可以聚合多个ByteBuffer对象，用户可以像操作一个Buffer那样方便的对组合Buffer进行操作，避免了传统通过内存拷贝的方式将几个小Buffer合并成一个大的Buffer。

> 3. Netty的文件传输采用了transferTo方法，它可以直接将文件缓冲区的数据发送到目标Channel，避免了传统通过循环write方式导致的内存拷贝问题。