当Sentinel集群有Sentinel发现master客观下线了，就会开始故障转移流程，故障转移流程的第一步就是在Sentinel集群选择一个Leader，让Leader完成故障转移流程。

### Raft协议选举流程

描述Raft选举流程之前需要了解一些概念。

#### 节点的状态

Raft协议描述的节点共有三种状态：Leader, Follower, Candidate。在系统运行正常的时候只有Leader和Follower两种状态的节点。一个Leader节点，其他的节点都是Follower。Candidate是系统运行不稳定时期的中间状态，当一个**Follower对Leader的的心跳出现异常，就会转变成Candidate**，Candidate会去竞选新的Leader，它会向其他节点发送竞选投票，如果大多数节点都投票给它，它就会替代原来的Leader，变成新的Leader，原来的Leader会降级成Follower。

#### term

在分布式系统中，各个节点的时间同步是一个很大的难题，但是为了识别过期时间，时间信息又必不可少。Raft协议为了解决这个问题，引入了term（任期）的概念。Raft协议将时间切分为一个个的Term，可以认为是一种“逻辑时间”。

#### RPC

Raft协议在选举阶段交互的RPC有两类：RequestVote和AppendEntries。

- RequestVote是用来向其他节点发送竞选投票。
- AppendEntries是当该节点得到更多的选票后，成为Leader，向其他节点确认消息。

Raft采用心跳机制触发Leader选举。系统启动后，全部节点初始化为Follower，term为0.节点如果收到了RequestVote或者AppendEntries，就会保持自己的Follower身份。如果一段时间内没收到AppendEntries消息直到选举超时，说明在该节点的超时时间内还没发现Leader，Follower就会转换成Candidate，自己开始竞选Leader。一旦转化为Candidate，该节点立即开始下面几件事情：

- 1、增加自己的term。
- 2、启动一个新的定时器。
- 3、给自己投一票。
- 4、向所有其他节点发送RequestVote，并等待其他节点的回复。

如果在这过程中收到了其他节点发送的AppendEntries，就说明已经有Leader产生，自己就转换成Follower，选举结束。

如果在计时器超时前，节点收到多数节点的同意投票，就转换成Leader。同时向所有其他节点发送AppendEntries，告知自己成为了Leader。

每个节点在一个term内只能投一票，采取先到先得的策略，Candidate前面说到已经投给了自己，Follower会投给第一个收到RequestVote的节点。每个Follower有一个计时器，在计时器超时时仍然没有接受到来自Leader的心跳RPC, 则自己转换为Candidate, 开始请求投票，就是上面的的竞选Leader步骤。

如果多个Candidate发起投票，每个Candidate都没拿到多数的投票（Split Vote），那么就会等到计时器超时后重新成为Candidate，重复前面竞选Leader步骤。

**Raft协议的定时器采取随机超时时间**，这是选举Leader的关键。每个节点定时器的超时时间随机设置，随机选取配置时间的1倍到2倍之间。由于随机配置，所以各个Follower同时转成Candidate的时间一般不一样，在同一个term内，先转为Candidate的节点会先发起投票，从而获得多数票。多个节点同时转换为Candidate的可能性很小。即使几个Candidate同时发起投票，在该term内有几个节点获得一样高的票数，只是这个term无法选出Leader。由于各个节点定时器的超时时间随机生成，那么最先进入下一个term的节点，将更有机会成为Leader。连续多次发生在一个term内节点获得一样高票数在理论上几率很小，实际上可以认为完全不可能发生。一般1-2个term类，Leader就会被选出来。

### Sentinel的选举流程

Sentinel集群正常运行的时候每个节点epoch相同，当需要故障转移的时候会在集群中选出Leader执行故障转移操作。Sentinel采用了Raft协议实现了Sentinel间选举Leader的算法，不过也不完全跟论文描述的步骤一致。Sentinel集群运行过程中故障转移完成，所有Sentinel又会恢复平等。Leader仅仅是故障转移操作出现的角色。

#### 选举流程

- 1、某个Sentinel认定master客观下线的节点后，该Sentinel会先看看自己有没有投过票，如果自己已经投过票给其他Sentinel了，在2倍故障转移的超时时间自己就不会成为Leader。相当于它是一个Follower。
- 2、如果该Sentinel还没投过票，那么它就成为Candidate。
- 3、和Raft协议描述的一样，成为Candidate，Sentinel需要完成几件事情 
  - 1）更新故障转移状态为start
  - 2）当前epoch加1，相当于进入一个新term，在Sentinel中epoch就是Raft协议中的term。
  - 3）更新自己的超时时间为当前时间随机加上一段时间，随机时间为1s内的随机毫秒数。
  - 4）向其他节点发送`is-master-down-by-addr`命令请求投票。命令会带上自己的epoch。
  - 5）给自己投一票，在Sentinel中，投票的方式是把自己master结构体里的leader和leader_epoch改成投给的Sentinel和它的epoch。
- 4、其他Sentinel会收到Candidate的`is-master-down-by-addr`命令。如果Sentinel当前epoch和Candidate传给他的epoch一样，说明他已经把自己master结构体里的leader和leader_epoch改成其他Candidate，相当于把票投给了其他Candidate。投过票给别的Sentinel后，在当前epoch内自己就只能成为Follower。
- 5、Candidate会不断的统计自己的票数，直到他发现认同他成为Leader的票数超过一半而且超过它配置的quorum（quorum可以参考[《redis sentinel设计与实现》](http://weizijun.cn/2015/04/30/redis sentinel 设计与实现/)）。Sentinel比Raft协议增加了quorum，这样一个Sentinel能否当选Leader还取决于它配置的quorum。
- 6、如果在一个选举时间内，Candidate没有获得超过一半且超过它配置的quorum的票数，自己的这次选举就失败了。
- 7、如果在一个epoch内，没有一个Candidate获得更多的票数。那么等待超过2倍故障转移的超时时间后，Candidate增加epoch重新投票。
- 8、如果某个Candidate获得超过一半且超过它配置的quorum的票数，那么它就成为了Leader。
- 9、与Raft协议不同，Leader并不会把自己成为Leader的消息发给其他Sentinel。其他Sentinel等待Leader从slave选出master后，检测到新的master正常工作后，就会去掉客观下线的标识，从而不需要进入故障转移流程。

#### 关于Sentinel超时时间的说明

Sentinel超时机制有几个超时概念。

- failover_start_time 下一选举启动的时间。默认是当前时间加上1s内的随机毫秒数
- failover_state_change_time 故障转移中状态变更的时间。
- failover_timeout 故障转移超时时间。默认是3分钟。
- election_timeout 选举超时时间，是默认选举超时时间和failover_timeout的最小值。默认是10s。

Follower成为Candidate后，会更新failover_start_time为当前时间加上1s内的随机毫秒数。更新failover_state_change_time为当前时间。

Candidate的当前时间减去failover_start_time大于election_timeout，说明Candidate还没获得足够的选票，此次epoch的选举已经超时，那么转变成Follower。需要等到`mstime() - failover_start_time < failover_timeout*2`的时候才开始下一次获得成为Candidate的机会。

如果一个Follower把某个Candidate设为自己认为的Leader，那么它的failover_start_time会设置为当前时间加上1s内的随机毫秒数。这样它就进入了上面说的需要等到`mstime() - failover_start_time < failover_timeout*2`的时候才开始下一次获得成为Candidate的机会。

因为每个Sentinel判断节点客观下线的时间不是同时开始的，一般都有先后，这样先开始的Sentinel就更有机会赢得更多选票，另外failover_state_change_time为1s内的随机毫秒数，这样也把各个节点的超时时间分散开来。本人尝试过很多次，Sentinel间的Leader选举过程基本上一个epoch内就完成了。

### Sentinel 选举流程源码解析

Sentinel的选举流程的代码基本都在sentinel.c文件中，下面结合源码对Sentinel的选举流程进行说明。