transport包下

# ChannelHandler

处理io事件或者拦截io操作,且转发到他的pipeline里的下一个处理器.

Handles an I/O event or intercepts an I/O operation, and forwards it to its next handler in its ChannelPipeline.
**Sub-types**
**ChannelHandler** itself does not provide many methods, but you usually have to implement one of its subtypes:
**ChannelInboundHandler** to handle inbound I/O events, and
**ChannelOutboundHandler** to handle outbound I/O operations.

Alternatively, the following adapter classes are provided for your convenience:

**ChannelInboundHandlerAdapter** to handle inbound I/O events,
**ChannelOutboundHandlerAdapter** to handle outbound I/O operations, and
**ChannelDuplexHandler** to handle both inbound and outbound events
For more information, please refer to the documentation of each subtype.
**The context object**
A ChannelHandler is provided with a ChannelHandlerContext object. A ChannelHandler is supposed to interact with the ChannelPipeline it belongs to via a context object. Using the context object, the ChannelHandler can pass events upstream or downstream, modify the pipeline dynamically, or store the information (using AttributeKeys) which is specific to the handler.

```java
void handlerAdded(ChannelHandlerContext ctx) throws Exception;
void handlerRemoved(ChannelHandlerContext ctx) throws Exception;
```

## ChannelInboundHandler

```java
//
void bind(ChannelHandlerContext ctx, SocketAddress localAddress, ChannelPromise promise) throws Exception;
//
void connect(
            ChannelHandlerContext ctx, SocketAddress remoteAddress,
            SocketAddress localAddress, ChannelPromise promise) throws Exception;
//
void disconnect(ChannelHandlerContext ctx, ChannelPromise promise) throws Exception;
//
void close(ChannelHandlerContext ctx, ChannelPromise promise) throws Exception;
//
void deregister(ChannelHandlerContext ctx, ChannelPromise promise) throws Exception;
//
void read(ChannelHandlerContext ctx) throws Exception;
//
void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception;
//
void flush(ChannelHandlerContext ctx) throws Exception;
```

## ChannelOutboundHandler

```java
//
void bind(ChannelHandlerContext ctx, SocketAddress localAddress, ChannelPromise promise) throws Exception;
//
void connect(
            ChannelHandlerContext ctx, SocketAddress remoteAddress,
            SocketAddress localAddress, ChannelPromise promise) throws Exception;
//
void disconnect(ChannelHandlerContext ctx, ChannelPromise promise) throws Exception;
//
void close(ChannelHandlerContext ctx, ChannelPromise promise) throws Exception;
//
void deregister(ChannelHandlerContext ctx, ChannelPromise promise) throws Exception;
//
void read(ChannelHandlerContext ctx) throws Exception;
//
void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception;
//
void flush(ChannelHandlerContext ctx) throws Exception;
```

可以看到入站出站的方法都是一样的,只是事件的流动方向是有区别的.

## ChannelHandlerContext 

Enables a ChannelHandler to interact with its ChannelPipeline and other handlers. Among other things a handler can notify the next ChannelHandler in the ChannelPipeline as well as modify the ChannelPipeline it belongs to dynamically.

类结构

```java
extends AttributeMap, ChannelInboundInvoker, ChannelOutboundInvoker {
   //	
   Channel channel();
	 //
   EventExecutor executor();
	 //
   String name();
   //
   boolean isRemoved();
   //
   ChannelHandlerContext fireChannelRegistered();
   //
   ChannelHandlerContext fireChannelUnregistered();
   //
   ChannelHandlerContext fireChannelActive();
   //
   ChannelHandlerContext fireChannelInactive();
   //
   ChannelHandlerContext fireExceptionCaught(Throwable cause);
   //
   ChannelHandlerContext fireUserEventTriggered(Object evt);
   //
   ChannelHandlerContext fireChannelRead(Object msg);
   //
   ChannelHandlerContext fireChannelReadComplete();
   //
   ChannelHandlerContext fireChannelWritabilityChanged();
   //
   ChannelHandlerContext read();
   //
   ChannelHandlerContext flush();
}
```

## Channel



## AbstractChannel

```java
//channel初始化的时候
protected AbstractChannel(Channel parent, ChannelId id) {
    this.parent = parent;
    this.id = id;
  	//初始化Unsafe
    unsafe = newUnsafe();
  	//初始化channelpipeline
    pipeline = newChannelPipeline();
}
```

初始化channelPipeline,默认的

```java
protected DefaultChannelPipeline newChannelPipeline() {
    return new DefaultChannelPipeline(this);
}
```

channelPipeline的初始化.

```java
protected DefaultChannelPipeline(Channel channel) {
  	//校验
    this.channel = ObjectUtil.checkNotNull(channel, "channel");
  	//future
    succeededFuture = new SucceededChannelFuture(channel, null);
  	//promise
    voidPromise =  new VoidChannelPromise(channel, true);
		//构建双向链表
    tail = new TailContext(this);
    head = new HeadContext(this);

    head.next = tail;
    tail.prev = head;
}
```

## 问题

channelPipeline什么时候被调用.

EchoServerHandler的channelRead是什么时候被调用的?

NioEventLoop.run里

然后processSelectedKeys

```java
  private void processSelectedKeys() {
        if (selectedKeys != null) {
            processSelectedKeysOptimized();
        } else {
            processSelectedKeysPlain(selector.selectedKeys());
        }
    }

```

然后

```java
processSelectedKey(k, (AbstractNioChannel) a);
```

然后:AbstractNioChannel.NioUnsafe

```java
unsafe.read();
```

然后:**已经调用到channelPipeline了.**

```java
pipeline.fireChannelRead(byteBuf);
```

