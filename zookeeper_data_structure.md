# zookeeper 数据结构

## StatPersisted

```
public class StatPersisted implements Record {
    long czxid;      // created zxid
    long mzxid;      // last modified zxid
    long ctime;      // created time
    long mtime;      // last modified
    int version;     // version
    int cversion;    // child version
    int aversion;    // acl version
    long ephemeralOwner; // owner id if ephemeral, 0 otw
    long pzxid;      // last modified children
}
```

持久化存储到磁盘的数据结构，详细的描述了一个znode节点的属性信息

这里的几个版本号中

- version 数据节点版本号，通过这个来控制实现乐观锁中的“写入校验”机制
- cversion 子节点版本号，znode子节点修改次数
- aversion znode访问控制列表的变化号

ephemeralOwner：不是临时节点时为0，是临时节点时为创建的sessionId

## DataNode

```
public class DataNode implements Record {

    // the digest value of this node, calculated from path, data and stat
    private volatile long digest;

    // indicate if the digest of this node is up to date or not, used to
    // optimize the performance.
    volatile boolean digestCached;

    /** the data for this datanode */
    byte[] data;

    /**
     * the acl map long for this datanode. the datatree has the map
     */
    Long acl;

    /**
     * the stat for this node that is persisted to disk.
     */
    public StatPersisted stat;

    /**
     * the list of children for this node. note that the list of children string
     * does not contain the parent path -- just the last part of the path. This
     * should be synchronized on except deserializing (for speed up issues).
     */
    private Set<String> children = null;

    private static final Set<String> EMPTY_SET = Collections.emptySet();
    ...
}
```

一个节点在内存中的表示，主要的属性有

- data 该节点存储的数据内容
- acl 访问权限的acl的id
- StatPersisted 持久化存储时的属性信息
- children 子节点路径，这里的子节点路径并不是全路径，而只是该节点的目录名

## NodeHashMapImpl

```
public class NodeHashMapImpl implements NodeHashMap {

    private final ConcurrentHashMap<String, DataNode> nodes;
    private final boolean digestEnabled;
    private final DigestCalculator digestCalculator;

    private final AdHash hash;
    ...
}
```

该数据结构是为了减少hash冲突，该结构体在zookeeper 3.6开始引入

- nodes key为完整的路径 value为一个节点的属性和数据信息
- digestEnabled为true的情况下表示开启了减少冲突的hash算法，会使用digestCalculator和hash计算hash值
- 该hash算法基于论文  A New Paradigm for collision-free hashing: Incrementality at reduced cost,  M. Bellare and D. Micciancio

## Stat

```
class Stat {
    long czxid;      // created zxid
    long mzxid;      // last modified zxid
    long ctime;      // created
    long mtime;      // last modified
    int version;     // version
    int cversion;    // child version
    int aversion;    // acl version
    long ephemeralOwner; // owner id if ephemeral, 0 otw
    int dataLength;  //length of the data in the node
    int numChildren; //number of children of this node
    long pzxid;      // last modified children
}
```

与客户端进行信息交互时的数据结构，可以看出其大部分与持久化的一样，唯一不同的是多了dataLength和numChildren两个字段

这两个字段是在zookeeper启动时构建起来的，在节点数据或者子节点个数变化时随着变换的

## FileHeader

```
class FileHeader {
    int magic;
    int version;
    long dbid;
}
```

进行持久化存储时的文件头， log文件和snapshot文件都有

## LearnerType

```
public enum LearnerType {
    PARTICIPANT,
    OBSERVER
}
```

在配置文件中可以配置服务器在集群中可以扮演的角色

- PARTICIPANT 服务器要参与选举，在选举过程中角色是LOOKING，选举完成后是FOLLOWING 或者LEADING
- OBSERVER 服务器不参与选举，只是状态保持跟leader一致，这种节点是为了提升读性能

## ServerState 

```
public enum ServerState {
    LOOKING,
    FOLLOWING,
    LEADING,
    OBSERVING
}
```

服务器可能的状态有四种

- LOOKING 刚启动正在选主过程中
- FOLLOWING 已经完成选举，属于跟随者角色
- LEADING 已经完成选举，属于领导角色
- OBSERVING 观察者角色，无论是否完成选举，都是观察者，我们没有使用该角色

