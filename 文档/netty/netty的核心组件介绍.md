## ChannelHandler

从应用程序开发人员的角度来看，Netty 的主要组件是 ChannelHandler，它充当了所有 处理入站和出站数据的应用程序逻辑的容器。

Handles an I/O event or intercepts an I/O operation, and forwards it to its next handler in its ChannelPipeline.

**The context object**
A ChannelHandler is provided with a ChannelHandlerContext object. A ChannelHandler is supposed to interact with the ChannelPipeline it belongs to via a context object. Using the context object, the ChannelHandler can pass events upstream or downstream, modify the pipeline dynamically, or store the information (using AttributeKeys) which is specific to the handler.

**State management**

## ChannelPipeline 接口

ChannelPipeline 提供了 ChannelHandler 链的容器，并定义了用于在该链上传播入站 和出站事件流的 API。当 Channel 被创建时，它会被自动地分配到它专属的 ChannelPipeline。

为了审查发送或者接收数据时将会发生什么，让我们来更加深入地研究 ChannelPipeline

和 ChannelHandler 之间的共生关系吧。

## ChannelHandlerContext





在Netty中，有两种发送消息的方式。你可以直接写到Channel中，也可以 写到和Channel- Handler 相关联的 ChannelHandlerContext 对象中。前一种方式将会导致消息从 Channel- Pipeline 的尾端开始流动，而后者将导致消息从 ChannelPipeline 中的下一个 Channel- Handler 开始流动。