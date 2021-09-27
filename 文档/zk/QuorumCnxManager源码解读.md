# 类文档

这个类用TCP为leader election实现了一个连接管理器.它为每对服务器维持了一个连接.棘手的是要保证没对服务器只有一个可以在网络上互相通讯的连接.如果两个服务器同时发起一次连接,那么连接管理器用一个非常简单的tie-breaking机制来决定放弃哪个连接.

# FLE是如何使用的,以及这个类的赋值过程

## WorkerReceiver里

```java
//取出一条消息
response = manager.pollRecvQueue(3000, TimeUnit.MILLISECONDS);
```

## WorkerSender

```java
manager.toSend(m.sid, requestBuffer);
```

# FLE

shutdwon里的halt

manager.haveDelivered()

manager.connectAll();



# 核心变量

```java
static final int RECV_CAPACITY = 100;
static final int SEND_CAPACITY = 1;
static final int PACKETMAXSIZE = 1024 * 512;
private AtomicLong observerCounter = new AtomicLong(-1);
public static final long PROTOCOL_VERSION = -65536L;
public static final int maxBuffer = 2048;
private int cnxTO = 5000;
final QuorumPeer self;
```

# 核心方法



# 内部类

## 1.SendWorker

### 1.1核心变量

```java
//发送给哪个服务器
Long sid;
//接受一个socket
Socket sock;
//持有一个RecvWorker,互相持有吧
RecvWorker recvWorker;
//是否是running
volatile boolean running = true;
//要发送的数据,RecvWorker肯定持有的是datainputstream
DataOutputStream dout;
```

## 2.RecvWorker



## 3.Listener



## 4.QuorumConnectionReceiverThread



## 5.QuorumConnectionReqThread