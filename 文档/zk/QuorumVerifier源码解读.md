类注释:

```java
/**
 * All quorum validators have to implement a method called
 * containsQuorum, which verifies if a HashSet of server
 * identifiers constitutes a quorum.
 * 所有的quorum validators必须实现containsQuorum这个方法.验证是否server的集合构成一个quorum
 * constitutes:构成
 */
```

```java
public interface QuorumVerifier {
		//获取权重
    long getWeight(long id);

    /**
     * Verifies if a set is a majority. Assumes that ackSet contains acks only
     * from votingMembers,ack.size > half
     * 验证一个set是否是majority.ack.size > half
     */
    boolean containsQuorum(Set<Long> set);
		//获取版本
    long getVersion();
		//设置版本
    void setVersion(long ver);
		//获取所有的成员
    Map<Long, QuorumServer> getAllMembers();
		//获取投票的成员
    Map<Long, QuorumServer> getVotingMembers();
		//获取观察的成员
    Map<Long, QuorumServer> getObservingMembers();
		//
    boolean equals(Object o);

    String toString();

}
```

实现类

```java
QuorumMaj
QuorumHierarchical
```

QuorumMaj

```java
public class QuorumMaj implements QuorumVerifier {
    //所有的成员
    private Map<Long, QuorumServer> allMembers = new HashMap<>();
    //投票的成员
    private Map<Long, QuorumServer> votingMembers = new HashMap<>();
    //观察的成员
    private Map<Long, QuorumServer> observingMembers = new HashMap<>();
    //版本
    private long version = 0;
    //过半机制
    private int half;

    //为什么返回42呢
    public int hashCode() {
        assert false : "hashCode not designed";
        return 42; // any arbitrary constant will do
    }

    public boolean equals(Object o) {
        if (!(o instanceof QuorumMaj)) {
            return false;
        }
        QuorumMaj qm = (QuorumMaj) o;
        if (qm.getVersion() == version) {
            return true;
        }
        if (allMembers.size() != qm.getAllMembers().size()) {
            return false;
        }
        for (QuorumServer qs : allMembers.values()) {
            QuorumServer qso = qm.getAllMembers().get(qs.id);
            if (qso == null || !qs.equals(qso)) {
                return false;
            }
        }
        return true;
    }

    /**
     * Defines a majority to avoid computing it every time.
     * 定义一个多数派,以避免每次都要计算
     * 这个allMembers肯定是从配置文件里获取到的
     */
    public QuorumMaj(Map<Long, QuorumServer> allMembers) {
        this.allMembers = allMembers;
        for (QuorumServer qs : allMembers.values()) {
            //两种身份,参与者还是观察者
            if (qs.type == LearnerType.PARTICIPANT) {
                votingMembers.put(Long.valueOf(qs.id), qs);
            } else {
                observingMembers.put(Long.valueOf(qs.id), qs);
            }
        }
        //投票者的一半
        half = votingMembers.size() / 2;
    }
		//传入属性键值对
    public QuorumMaj(Properties props) throws ConfigException {
        for (Entry<Object, Object> entry : props.entrySet()) {
            String key = entry.getKey().toString();
            String value = entry.getValue().toString();

            if (key.startsWith("server.")) {
                int dot = key.indexOf('.');
              	//截出sid
                long sid = Long.parseLong(key.substring(dot + 1));
              	//
                QuorumServer qs = new QuorumServer(sid, value);
                allMembers.put(Long.valueOf(sid), qs);
                if (qs.type == LearnerType.PARTICIPANT) {
                    votingMembers.put(Long.valueOf(sid), qs);
                } else {
                    observingMembers.put(Long.valueOf(sid), qs);
                }
            } else if (key.equals("version")) {
                version = Long.parseLong(value, 16);
            }
        }
      	//保存一半
        half = votingMembers.size() / 2;
    }

    /**
     * Returns weight of 1 by default.
     *
     * @param id
     */
    public long getWeight(long id) {
        return 1;
    }

    public String toString() {
        StringBuilder sw = new StringBuilder();

        for (QuorumServer member : getAllMembers().values()) {
            String key = "server." + member.id;
            String value = member.toString();
            sw.append(key);
            sw.append('=');
            sw.append(value);
            sw.append('\n');
        }
        String hexVersion = Long.toHexString(version);
        sw.append("version=");
        sw.append(hexVersion);
        return sw.toString();
    }

    /**
     * Verifies if a set is a majority. Assumes that ackSet contains acks only
     * from votingMembers
     */
    public boolean containsQuorum(Set<Long> ackSet) {
        return (ackSet.size() > half);
    }

    public Map<Long, QuorumServer> getAllMembers() {
        return allMembers;
    }

    public Map<Long, QuorumServer> getVotingMembers() {
        return votingMembers;
    }

    public Map<Long, QuorumServer> getObservingMembers() {
        return observingMembers;
    }

    public long getVersion() {
        return version;
    }

    public void setVersion(long ver) {
        version = ver;
    }

}
```

```java
 public enum LearnerType {
   			//参与者
        PARTICIPANT,
   			//观察者
        OBSERVER
    }
```

