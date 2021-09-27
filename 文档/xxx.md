## NioEventLoop

### doBind

### initAndRegister

### init

### io.netty.bootstrap.ServerBootstrap#init



## EventExecutor

类结构

## ChannelFuture ChannelFutureListener

The result of an asynchronous Channel I/O operation.

## 关系

Channel,EventLoop,Thread,EventLoopGroup直接的关系.

1.一个 EventLoopGroup 包含一个或者多个 EventLoop;

2.一个 EventLoop 在它的生命周期内只和一个 Thread 绑定;

3.所有由 EventLoop 处理的 I/O 事件都将在它专有的 Thread 上被处理;

4.一个 Channel 在它的生命周期内只注册于一个 EventLoop;

5.一个 EventLoop 可能会被分配给一个或多个 Channel。

在这种设计中,一个给定Channel的I/O操作都是由相同的Thread执行的,实际上消除了对同步的需要.

## **ChannelHandler** ChannelPipeline

应用程序开发人员的角度来看,netty的主要组件是ChannelHandler,它充当了所有处理入站和出站数据的应用程序逻辑的容器.

ChannelPipeline:提供了ChannelHandler链的容器.channel

使得事件流经 ChannelPipeline 是 ChannelHandler 的工作，它们是在应用程序的初 始化或者引导阶段被安装的。

如果事件的运动方向是从客户端到服务器端，那么我们称这些事件为出站的，反之 则称为入站的。

传输api的核心是interface Channel.

## netty是如何统一api设计的.

Intercepting Filter.

netty如何使用epoll.

零拷贝:

## EpollEventLoopGroup



## ByteBuf

![image-20210123184646288](/Users/lumac/Library/Application Support/typora-user-images/image-20210123184646288.png)

netty的数据容器

读写双索引.

#### 1.堆缓冲区

hasArray

#### 2.直接缓冲区

NIO 在 JDK 1.4 中引入的 ByteBuffer 类允许 JVM 实现通过本地调 用来分配内存。这主要是为了避免在每次调用本地 I/O 操作之前(或者之后)将缓冲区的内容复 制到一个中间缓冲区(或者从中间缓冲区把内容复制到缓冲区)。

直接缓冲区的主要缺点:相对于基于堆的缓冲区，它们的分配和释放都较为昂贵

#### 3.复合缓冲区

CompositeByteBuf

需要注意的是，Netty使用了CompositeByteBuf来优化套接字的I/O操作，尽可能地消除了 由JDK的缓冲区实现所导致的性能以及内存使用率的惩罚。1这种优化发生在Netty的核心代码中， 因此不会被暴露出来，但是你应该知道它所带来的影响。

#### 顺序访问索引

虽然 ByteBuf 同时具有读索引和写索引，但是 JDK 的 ByteBuffer 却只有一个索引，这 也就是为什么必须调用 flip()方法来在读模式和写模式之间进行切换的原因。

![image-20210123193718958](/Users/lumac/Library/Application Support/typora-user-images/image-20210123193718958.png)

虽然你可能会倾向于频繁地调用 discardReadBytes()方法以确保可写分段的最大化，但是 请注意，这将极有可能会导致内存复制，因为可读字节(图中标记为 CONTENT 的部分)必须被移 动到缓冲区的开始位置。我们建议只在有真正需要的时候才这样做，例如，当内存非常宝贵的时候。

## ReferenceCounted



## netty里的零拷贝



## ChannelHandlerContext

ChannelPipeline,ChannelHandler,ChannelHandlerContext

理解所有这些组件之间的交互对于通过 Netty 构建模块化的、可重用的实现至关重要。

Channel的四个状态:

1.ChannelUnregistered

2.ChannelRegistered

3.ChannelActive

4.ChannelInactive

## ChannelHandler 的生命周期

ChannelInboundHandler



ChannelOutboundHandler

## 资源管理

## 线程模型

![image-20210123214900091](/Users/lumac/Library/Application Support/typora-user-images/image-20210123214900091.png)