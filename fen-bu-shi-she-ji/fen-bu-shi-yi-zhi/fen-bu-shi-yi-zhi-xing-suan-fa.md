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
  * 解决方案：记录日志+补偿（正向补充或者回滚）、消息重试\(要求程序要幂等\)；“无锁设计”、采用乐观锁机制。



