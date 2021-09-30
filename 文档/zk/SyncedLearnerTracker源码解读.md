这个类还是比较简单的.

核心变量:QuorumVerifierAcksetPair这个类在本文的最下面

```java
protected final ArrayList<QuorumVerifierAcksetPair> qvAcksetPairs = new ArrayList<>();
```

```java
//QuorumVerifierAcksetPair的集合
//QuorumVerifierAcksetPair,qv对应的ack信息
//看看QuorumVerifier的赋值情况
//QuorumVerifier的赋值情况
//QuorumPeer self;self.getQuorumVerifier()
//然后查看self是如何赋值的.self是通过FastLeaderElection(QuorumPeer self, QuorumCnxManager manager)
//FastLeaderElection fle = new FastLeaderElection(this, qcm);
//QuorumPeer中调用FLT的构造器,那么QuorumPeer显然是在createElectionAlgorithm这个方法里生成的
//构造器里进行赋值QuorumPeer()
// if (quorumConfig == null) {
//            quorumConfig = new QuorumMaj(quorumPeers);
//        }
//        setQuorumVerifier(quorumConfig, false);
//最好从入口再重新观察下各个重要参数的赋值情况
public void addQuorumVerifier(QuorumVerifier qv) {
    qvAcksetPairs.add(new QuorumVerifierAcksetPair(qv, new HashSet<>(qv.getVotingMembers().size())));
}
```

```java
//实际上这个返回值是没有被用到的,只是单纯的修改了qvAckset
//把ack的sid添加到ackset里面
public boolean addAck(Long sid) {
    boolean change = false;
    for (QuorumVerifierAcksetPair qvAckset : qvAcksetPairs) {
        //如果投票的sid包含目标sid
        if (qvAckset.getQuorumVerifier().getVotingMembers().containsKey(sid)) {
            //
            qvAckset.getAckset().add(sid);
            change = true;
        }
    }
    return change;
}
//qvAcksetPairs的每个qvAckset都要包含sid,才是真,否则为false
public boolean hasSid(long sid) {
    for (QuorumVerifierAcksetPair qvAckset : qvAcksetPairs) {
        if (!qvAckset.getQuorumVerifier().getVotingMembers().containsKey(sid)) {
            return false;
        }
    }
    return true;
}
//是否包含
public boolean hasAllQuorums() {
    for (QuorumVerifierAcksetPair qvAckset : qvAcksetPairs) {
        //
        if (!qvAckset.getQuorumVerifier().containsQuorum(qvAckset.getAckset())) {
            return false;
        }
    }
    return true;
}

public String ackSetsToString() {
    StringBuilder sb = new StringBuilder();

    for (QuorumVerifierAcksetPair qvAckset : qvAcksetPairs) {
        sb.append(qvAckset.getAckset().toString()).append(",");
    }

    return sb.substring(0, sb.length() - 1);
}
```

内部类:单纯的数据封装类

```java
public static class QuorumVerifierAcksetPair {
    //QuorumVerifier有哪些信息呢,看看这个实现类QuorumMaj
    private final QuorumVerifier qv;
    //ack集合,看下从哪里赋值的:qv.getVotingMembers().size()
    private final HashSet<Long> ackset;

    public QuorumVerifierAcksetPair(QuorumVerifier qv, HashSet<Long> ackset) {
        this.qv = qv;
        this.ackset = ackset;
    }

    public QuorumVerifier getQuorumVerifier() {
        return this.qv;
    }

    public HashSet<Long> getAckset() {
        return this.ackset;
    }
```

