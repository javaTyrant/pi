# Leader源码解读

顾名思义,leader这个类最主要的算法应该是lead,当一个服务器被选举为lead时,这个类的lead方法就会被调用.

Leader是继承了LearnerMaster这个抽象类.而这个类是为了让观察者保持同步的方法.

## 1.1核心变量

```java
//tcp no delay
private static final boolean nodelay = System.getProperty("leader.nodelay", "true").equals("true")
// log ack latency if zxid is a multiple of ackLoggingFrequency. If <=0, disable logging.
private static final String ACK_LOGGING_FREQUENCY = "zookeeper.leader.ackLoggingFrequency";
//akc打日志的频率,提供get,set方法
private static int ackLoggingFrequency;
//leader服务,关注下和zookeepserver的区别
final LeaderZooKeeperServer zk;
//
final QuorumPeer self;
//
protected boolean quorumFormed = false;
//注意volatile的
volatile LearnerCnxAcceptor cnxAcceptor = null;
//
private final HashSet<LearnerHandler> learners = new HashSet<>();
//
private final BufferStats proposalStats;
```



## 1.2构造器

## 1.3关键方法

#### 1.3.1.lead方法无疑是最最核心的方法了

```java
* lead做了哪几件事呢
* 1.接收 follower 连接
* 2.计算新的 epoch 值
* 3.通知统一 epoch 值
* 4.数据同步
* 5.启动 zk server 对外提供服务
```

```java
 void lead() throws IOException, InterruptedException {
  	//选举结束的时间
   	self.end_fle = Time.currentElapsedTime();
    //看看这个名字取得多好,taken,学到了	
    long electionTimeTaken = self.end_fle - self.start_fle;
   	//选举结束这两个值清空
   	self.start_fle = 0;
    self.end_fle = 0;
   	//注册jmx
    zk.registerJMX(new LeaderBean(this, zk), self.jmxLocalPeerBean);
    //
 }
```

#### 1.3.2.processAck



#### 1.3.3.processSync



#### 1.3.4.propose

```java

```



## 1.4内部类

### 1.4.1 LearnerCnxAcceptor



### 1.4.2 Proposal

```java
Proposal extends SyncedLearnerTracker
//数据包:type
public QuorumPacket packet;      
//请求:zxid
public Request request;
@Override
 public String toString() {       
      return packet.getType() + ", " + packet.getZxid() + ", " + request;      
 }  
       
```

### 1.4.3 ToBeAppliedRequestProcessor



### 1.4.4 XidRolloverException