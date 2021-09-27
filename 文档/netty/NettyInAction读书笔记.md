## 3.netty的组件和设计

在我们更加详细地研究 Netty 的各个组件时，我们将密切关注它们是如何通过协作来支撑这 些体系结构上的最佳实践的。

Channel -> Socket

Eventloop -> 控制流,多线程处理,并发

ChannelFuture->异步通知

### Channel接口



### EventLoop接口



### ChannelHandler 安装到 ChannelPipeline 中的过程

- 一个ChannelInitializer的实现被注册到了ServerBootstrap中 1;

- 当 ChannelInitializer.initChannel()方法被调用时，ChannelInitializer,将在 ChannelPipeline 中安装一组自定义的 ChannelHandler;

- ChannelInitializer 将它自己从 ChannelPipeline 中移除。

