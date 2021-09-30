生活场景:

饭店规模的变化

迎宾,点菜,做饭,上菜,打扫

## Reactor模型

什么是reactor?

核心流程:

注册感兴趣的事件->扫描是否有感兴趣的事件发生->事件发生后做出相应的处理

![image-20210203212453749](/Users/lumac/Library/Application Support/typora-user-images/image-20210203212453749.png)

![image-20210203212525403](/Users/lumac/Library/Application Support/typora-user-images/image-20210203212525403.png)

![image-20210203212944850](/Users/lumac/Library/Application Support/typora-user-images/image-20210203212944850.png)

![image-20210203213027246](/Users/lumac/Library/Application Support/typora-user-images/image-20210203213027246.png)

![image-20210203213046690](/Users/lumac/Library/Application Support/typora-user-images/image-20210203213046690.png)

四个问题:

1.netty如何支持reactor模式的

2.为什么netty的main reactor大多并不能用到一个线程组,只能用到线程组里的一个呢?

3.netty给channel分配nio eventloop的规则是什么.

4.通用模式的nio实现多路复用是怎么跨平台的.

5.socketchannel和group的绑定关系,源码级别的吃透.

EventExecutorChooser.

