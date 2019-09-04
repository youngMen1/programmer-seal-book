## 分布式一致

### CAP 与 BASE 理论

* [《从分布式一致性谈到CAP理论、BASE理论》](http://www.cnblogs.com/szlbm/p/5588543.html)
  * 一致性分类：强一致\(立即一致\)；弱一致\(可在单位时间内实现一致，比如秒级\)；最终一致\(弱一致的一种，一定时间内最终一致\)
  * CAP：一致性、可用性、分区容错性\(网络故障引起\)
  * BASE：Basically Available（基本可用）、Soft state（软状态）和Eventually consistent（最终一致性）
  * BASE理论的核心思想是：即使无法做到强一致性，但每个应用都可以根据自身业务特点，采用适当的方式来使系统达到最终一致性。

### 分布式锁

* [《分布式锁的几种实现方式》](http://www.hollischuang.com/archives/1716)

  * 基于数据库的分布式锁：优点：操作简单、容易理解。缺点：存在单点问题、数据库性能够开销较大、不可重入；
  * 基于缓存的分布式锁：优点：非阻塞、性能好。缺点：操作不好容易造成锁无法释放的情况。
  * Zookeeper 分布式锁：通过有序临时节点实现锁机制，自己对应的节点需要最小，则被认为是获得了锁。优点：集群可以透明解决单点问题，避免锁不被释放问题，同时锁可以重入。缺点：性能不如缓存方式，吞吐量会随着zk集群规模变大而下降。

* [《基于Zookeeper的分布式锁》](https://www.tuicool.com/articles/VZJr6fY)

  * 清楚的原理描述 + Java 代码示例。

* [《jedisLock—redis分布式锁实现》](https://www.cnblogs.com/0201zcr/p/5942748.html)

  * 基于 setnx\(set if ont exists\)，有则返回false，否则返回true。并支持过期时间。

* [《Memcached 和 Redis 分布式锁方案》](https://blog.csdn.net/albertfly/article/details/77412333)

  * 利用 memcached 的 add（有别于set）操作，当key存在时，返回false。

### 分布式一致性算法

#### PAXOS

* [《分布式系列文章——Paxos算法原理与推导》](https://www.cnblogs.com/linbingdong/p/6253479.html)
* [《Paxos--&gt;Fast Paxos--&gt;Zookeeper分析》](https://blog.csdn.net/u010039929/article/details/70171672)
* [《【分布式】Zookeeper与Paxos》](https://www.cnblogs.com/leesf456/p/6012777.html)

#### Zab

* [《Zab：Zookeeper 中的分布式一致性协议介绍》](https://www.jianshu.com/p/fb527a64deee)

#### Raft

* [《Raft 为什么是更易理解的分布式一致性算法》](http://www.cnblogs.com/mindwind/p/5231986.html)
  * 三种角色：Leader（领袖）、Follower（群众）、Candidate（候选人）
  * 通过随机等待的方式发出投票，得票多的获胜。

#### Gossip

* [《Gossip算法》](http://blog.51cto.com/tianya23/530743)

#### 两阶段提交、多阶段提交

* [《关于分布式事务、两阶段提交协议、三阶提交协议》](http://blog.jobbole.com/95632/)

### 幂等

* [《分布式系统---幂等性设计》](https://www.cnblogs.com/wxgblogs/p/6639272.html)
  * 幂等特性的作用：该资源具备幂等性，请求方无需担心重复调用会产生错误。
  * 常见保证幂等的手段：MVCC（类似于乐观锁）、去重表\(唯一索引\)、悲观锁、一次性token、序列号方式。

### 分布式一致方案

* [《分布式系统事务一致性解决方案》](http://www.infoq.com/cn/articles/solution-of-distributed-system-transaction-consistency)
* [《保证分布式系统数据一致性的6种方案》](https://weibo.com/ttarticle/p/show?id=2309403965965003062676)

### 分布式 Leader 节点选举

* [《利用zookeeper实现分布式leader节点选举》](https://blog.csdn.net/johnson_moon/article/details/78809995)

### TCC\(Try/Confirm/Cancel\) 柔性事务

* [《传统事务与柔性事务》](https://www.jianshu.com/p/ab1a1c6b08a1)
  * 基于BASE理论：基本可用、柔性状态、最终一致。
  * 解决方案：记录日志+补偿（正向补



