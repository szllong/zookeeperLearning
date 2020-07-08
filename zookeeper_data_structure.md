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

zookeeper进行持久化的数据结构，详细的描述了一个znode节点的属性信息

这里的几个版本号中

- version 数据节点版本号，通过这个来控制实现乐观锁中的“写入校验”机制
- cversion 子节点版本号，znode子节点修改次数
- aversion znode访问控制列表的变化号

ephemeralOwner：不是临时节点时为0，是临时节点时为创建的sessionId

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

