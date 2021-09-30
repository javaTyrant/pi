# [Eureka的工作原理以及它与ZooKeeper的区别](https://www.cnblogs.com/linjiqin/p/10080444.html)

Eureka 是 Netflix 出品的用于实现服务注册和发现的工具。 Spring Cloud 集成了 Eureka，并提供了开箱即用的支持。其中， Eureka 又可细分为 Eureka Server 和 Eureka Client。

![img](https://img2018.cnblogs.com/blog/270324/201812/270324-20181206234223025-1241134471.png)

2**、基本原理**
上图来自eureka的官方架构图，这是基于集群配置的eureka；
\- 处于不同节点的eureka通过Replicate进行数据同步
\- Application Service为服务提供者
\- Application Client为服务消费者
\- Make Remote Call完成一次服务调用

服务启动后向Eureka注册，Eureka Server会将注册信息向其他Eureka Server进行同步，当服务消费者要调用服务提供者，则向服务注册中心获取服务提供者地址，然后会将服务提供者地址缓存在本地，下次再调用时，则直接从本地缓存中取，完成一次调用。

当服务注册中心Eureka Server检测到服务提供者因为宕机、网络原因不可用时，则在服务注册中心将服务置为DOWN状态，并把当前服务提供者状态向订阅者发布，订阅过的服务消费者更新本地缓存。

服务提供者在启动后，周期性（默认30秒）向Eureka Server发送心跳，以证明当前服务是可用状态。Eureka Server在一定的时间（默认90秒）未收到客户端的心跳，则认为服务宕机，注销该实例。(心跳)

**3、Eureka的自我保护机制**

在默认配置中，Eureka Server在默认90s没有得到客户端的心跳，则注销该实例，但是往往因为微服务跨进程调用，网络通信往往会面临着各种问题，比如微服务状态正常，但是因为网络分区故障时，Eureka Server注销服务实例则会让大部分微服务不可用，这很危险，因为服务明明没有问题。

为了解决这个问题，Eureka 有自我保护机制，通过在Eureka Server配置如下参数，可启动保护机制
eureka.server.enable-self-preservation=true

它的原理是，当Eureka Server节点在短时间内丢失过多的客户端时（可能发送了网络故障），那么这个节点将进入自我保护模式，不再注销任何微服务，当网络故障回复后，该节点会自动退出自我保护模式。

自我保护模式的架构哲学是宁可放过一个，决不可错杀一千

**4、作为服务注册中心，Eureka比Zookeeper好在哪里**

著名的CAP理论指出，一个分布式系统不可能同时满足C(一致性)、A(可用性)和P(分区容错性)。由于分区容错性在是分布式系统中必须要保证的，因此我们只能在A和C之间进行权衡。在此Zookeeper保证的是CP, 而Eureka则是AP。

4.1 Zookeeper保证CP
当向注册中心查询服务列表时，**我们可以容忍注册中心返回的是几分钟以前的注册信息，但不能接受服务直接down掉不可用。**也就是说，**服务注册功能对可用性的要求要高于一致性**。但是zk会出现这样一种情况，当master节点因为网络故障与其他节点失去联系时，剩余节点会重新进行leader选举。**问题在于，选举leader的时间太长，30 ~ 120s, 且选举期间整个zk集群都是不可用的，这就导致在选举期间注册服务瘫痪。**在云部署的环境下，因网络问题使得zk集群失去master节点是较大概率会发生的事，虽然服务能够最终恢复，但是漫长的选举时间导致的注册长期不可用是不能容忍的。

1. Eureka不再从注册列表中移除因为长时间没收到心跳而应该过期的服务
2. Eureka仍然能够接受新服务的注册和查询请求，但是不会被同步到其它节点上(即保证当前节点依然可用)
3. 当网络稳定时，当前实例新的注册信息会被同步到其它节点中

因此，Eureka可以很好的应对因网络故障导致部分节点失去联系的情况，而不会像zookeeper那样使整个注册服务瘫痪。

**5、总结**
Eureka作为单纯的服务注册中心来说要比zookeeper更加“专业”，因为注册服务更重要的是可用性，我们可以接受短期内达不到一致性的状况。不过Eureka目前1.X版本的实现是基于servlet的Java web应用，它的极限性能肯定会受到影响。期待正在开发之中的2.X版本能够从servlet中独立出来成为单独可部署执行的服务。

## Eureka数据同步过程

分布式系统的数据在多个副本之间的复制方式，主要有：

- 主从复制

就是 **Master-Slave** 模式，有一个主副本，其他为从副本，所有写操作都提交到主副本，再由主副本更新到其他从副本。

写压力都集中在主副本上，是系统的瓶颈，从副本可以分担读请求。

- 对等复制

就是 **Peer to Peer** 模式，副本间不分主从，任何副本都可以接收写操作，然后每个副本间互相进行数据更新。

对等复制模式，任何副本都可以接收写请求，不存在写压力瓶颈，但各个副本间数据同步时可能产生数据冲突。

Eureka 采用的就是 **Peer to Peer** 模式。

#### **2.2 同步过程**

Eureka Server 本身依赖了 Eureka Client，也就是每个 Eureka Server 是作为其他 Eureka Server 的 Client。

Eureka Server 启动后，会通过 Eureka Client 请求其他 Eureka Server 节点中的一个节点，获取注册的服务信息，然后复制到其他 peer 节点。

Eureka Server 每当自己的信息变更后，例如 Client 向自己发起*注册、续约、注销*请求， 就会把自己的最新信息通知给其他 Eureka Server，保持数据同步。

![img](https://user-gold-cdn.xitu.io/2019/12/9/16eeaced7e0ba544?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

如果自己的信息变更是另一个Eureka Server同步过来的，这是再同步回去的话就出现**数据同步死循环**了。

![img](https://user-gold-cdn.xitu.io/2019/12/9/16eeaced7f794e92?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Eureka Server 在执行复制操作的时候，使用 `HEADER_REPLICATION` 这个 http header 来区分普通应用实例的正常请求，说明这是一个复制请求，这样其他 peer 节点收到请求时，就不会再对其进行复制操作，从而避免死循环。

还有一个问题，就是**数据冲突**，比如 server A 向 server B 发起同步请求，如果 A 的数据比 B 的还旧，B 不可能接受 A 的数据，那么 B 是如何知道 A 的数据是旧的呢？这时 A 又应该怎么办呢？

数据的新旧一般是通过*版本号*来定义的，Eureka 是通过 `lastDirtyTimestamp` 这个类似版本号的属性来实现的。

> `lastDirtyTimestamp` 是注册中心里面服务实例的一个属性，表示此服务实例最近一次变更时间。

比如 Eureka Server A 向 Eureka Server B 复制数据，数据冲突有2种情况：

（1）A 的数据比 B 的新，B 返回 404，A 重新把这个应用实例注册到 B。

（2）A 的数据比 B 的旧，B 返回 409，要求 A 同步 B 的数据。


![img](https://user-gold-cdn.xitu.io/2019/12/9/16eeaced7f270434?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

还有一个重要的机制：**hearbeat 心跳**，即续约操作，来进行数据的最终修复，因为节点间的复制可能会出错，通过心跳就可以发现错误，进行弥补。

例如发现某个应用实例数据与某个server不一致，则server反回404，实例重新注册即可。

### eureka组件调用关系

**服务提供者**

1. 启动后，向注册中心发起 **register** 请求，注册服务
2. 在运行过程中，定时向注册中心发送 **renew** 心跳，证明“我还活着”。
3. 停止服务提供者，向注册中心发起 **cancel** 请求，清空当前服务注册信息。

**服务消费者**

1. 启动后，从注册中心拉取服务注册信息
2. 在运行过程中，定时更新服务注册信息。
3. **服务消费者发起远程调用：**
4. a> 服务消费者（北京）会从服务注册信息中选择同机房的服务提供者（北京），发起远程调用。只有同机房的服务提供者挂了才会选择其他机房的服务提供者（青岛）。
5. b> 服务消费者（天津）因为同机房内没有服务提供者，则会按负载均衡算法选择北京或青岛的服务提供者，发起远程调用。

**注册中心**

1. 启动后，从其他节点拉取服务注册信息。
2. 运行过程中，定时运行 **evict** 任务，剔除没有按时 **renew** 的服务（包括非正常停止和网络故障的服务）。
3. 运行过程中，接收到的 register、renew、cancel 请求，都会同步至其他注册中心节点。
4.  
5. 本文将详细说明上图中的 registry、register、renew、cancel、getRegistry、evict 的内部机制。

## 数据存储结构

既然是服务注册中心，必然要存储服务的信息，我们知道 ZK 是将服务信息保存在树形节点上。而下面是 Eureka 的数据存储结构：

![img](https://static001.infoq.cn/resource/image/61/2a/61a26fffaeaf63d5dd5c0c2c5dd7852a.png)

Eureka 的数据存储分了两层：数据存储层和缓存层。

Eureka Client 在拉取服务信息时，先从缓存层获取（相当于 Redis），如果获取不到，先把数据存储层的数据加载到缓存中（相当于 Mysql），再从缓存中获取。值得注意的是，数据存储层的数据结构是服务信息，而缓存中保存的是经过处理加工过的、可以直接传输到 Eureka Client 的数据结构。

Eureka 这样的数据结构设计是把内部的数据存储结构与对外的数据结构隔离开了，就像是我们平时在进行接口设计一样，对外输出的数据结构和数据库中的数据结构往往都是不一样的。

**数据存储层**

这里为什么说是存储层而不是持久层？因为 rigistry 本质上是一个双层的 ConcurrentHashMap，存储在内存中的。



- 第一层的 key 是`spring.application.name`，value 是第二层 ConcurrentHashMap；
- 第二层 ConcurrentHashMap 的 key 是服务的 InstanceId，value 是 Lease 对象；
- Lease 对象包含了服务详情和服务治理相关的属性。
-  
- **二级缓存层**

Eureka 实现了二级缓存来保存即将要对外传输的服务信息，数据结构完全相同。

- 一级缓存：`ConcurrentHashMap<Key,Value> readOnlyCacheMap`，本质上是 HashMap，无过期时间，保存服务信息的对外输出数据结构。
- 二级缓存：`Loading<Key,Value> readWriteCacheMap`，本质上是 guava 的缓存，包含失效机制，保存服务信息的对外输出数据结构。

既然是缓存，那必然要有更新机制，来保证数据的一致性。下面是缓存的更新机制：

![img](https://static001.infoq.cn/resource/image/2d/c3/2dcf60f2fa8ec888f2b3262e6e9de5c3.png)

更新机制包含删除和加载两个部分，上图黑色箭头表示删除缓存的动作，绿色表示加载或触发加载的动作。

**删除二级缓存：**

1. Eureka Client 发送 register、renew 和 cancel 请求并更新 registry 注册表之后，删除二级缓存；
2. Eureka Server 自身的 Evict Task 剔除服务后，删除二级缓存；
3. 二级缓存本身设置了 guava 的失效机制，隔一段时间后自己自动失效；

**加载二级缓存：**

1. Eureka Client 发送 getRegistry 请求后，如果二级缓存中没有，就触发 guava 的 load，即从 registry 中获取原始服务信息后进行处理加工，再加载到二级缓存中。
2. Eureka Server 更新一级缓存的时候，如果二级缓存没有数据，也会触发 guava 的 load。

**更新一级缓存：**

1. Eureka Server 内置了一个 TimerTask，定时将二级缓存中的数据同步到一级缓存（这个动作包括了删除和加载）。
2.  d
3. 关于缓存的实现参考 **ResponseCacheImpl**