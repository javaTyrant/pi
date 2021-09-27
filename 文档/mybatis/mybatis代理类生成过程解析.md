- 1、 Mapper接口是如何被加载 到 Configuration 中的？
- 2、 Mapper代理类是如何生成的？
- 3、 Mapper代理类是如何实现接口方法的？

### 一、加载 Mapper接口



### 二、 Mapper动态代理对象（MapperProxy）的创建



### 三、 MapperProxy 接口方法的实现



### 四、 个人总结

- 1、 Configuration 维护了一个 **MapperRegistry** 对象，该对象主要作用就是加载Mapper接口和获取MapperProxy。
- 2、 MapperRegistry 维护了一个key为 mapper接口class对象，value为 **MapperProxyFactory** 的map
- 3、 **MapperProxy** 是 通过  **MapperProxyFactory** 创建的。
- 4、 **MapperProxy** 实现 Mapper接口方法是委托 **MapperMethod** 执行的。
- 5、 **MapperMethod** 执行接口方法时是通过 **SqlCommand** 来判断要执行的具体 SQL节点，并且最终委托 **SqlSession**执行。
- 6、 **SqlCommand** 内部的 信息是通过从 **MappedStatement** 中获取的。

