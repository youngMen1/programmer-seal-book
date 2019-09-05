# Zookeeper 中的分布式一致性协议介绍

# 背景

在分布式系统中实现一致性是件有挑战的事。经典的二阶段提交、三阶段提交都不能完美的解决这一问题，有关传统的的分布式系统一致性问题可以看[这里](https://link.jianshu.com?t=http://coolshell.cn/articles/10910.html)。[Paxos](https://link.jianshu.com?t=https://en.wikipedia.org/wiki/Paxos_%28computer_science%29) 算法能完美地达到分布式系统的一致性，但由于较为复杂，在实际工程上不是很合适，Zab 协议借鉴了 Paxos 的思想，并进行了改进，以满足工程上的实际需求。

# 设计目标

* 一致性
* 有序性：有序性是 Zab 协议与 Paxos 协议的一个核心区别。Zab 的有序性主要表现在两个方面：
  1. 全局有序：如果消息 a 在消息 b 之前被投递，那么在任何一台服务器，消息 a都会在消息 b 之前被投递。
  2. 因果有序：如果消息 a 在消息 b 之前发生（a 导致了 b），并被一起发送，则 a 始终在 b 之前被执行。
* 容错性：有 2f+1 台服务器，只要有大于等于 f+1 台的服务器正常工作，就能完全正常工作。

# 协议内容

Zab 协议分为两大块：

* 广播（boardcast）：Zab 协议中，所有的写请求都由 leader 来处理。正常工作状态下，leader 接收请求并通过广播协议来处理。

* 恢复（recovery）：当服务初次启动，或者 leader 节点挂了，系统就会进入恢复模式，直到选出了有合法数量 follower 的新 leader，然后新 leader 负责将整个系统同步到最新状态。

### 广播（boardcast）

广播的过程实际上是一个简化的二阶段提交过程：

1. Leader 接收到消息请求后，将消息赋予一个全局唯一的 64 位自增 id，叫做：zxid，通过 zxid 的大小比较即可实现因果有序这一特性。
2. Leader 通过先进先出队列（通过 TCP 协议来实现，以此实现了全局有序这一特性）将带有 zxid 的消息作为一个提案（proposal）分发给所有 follower。
3. 当 follower 接收到 proposal，先将 proposal 写到硬盘，写硬盘成功后再向 leader 回一个 ACK。
4. 当 leader 接收到合法数量的 ACKs 后，leader 就向所有 follower 发送 COMMIT 命令，同事会在本地执行该消息。
5. 当 follower 收到消息的 COMMIT 命令时，就会执行该消息

2717543-03f77d9b25184821.webp

相比于完整的二阶段提交，Zab 协议最大的区别就是不能终止事务，follower 要么回 ACK 给 leader，要么抛弃 leader，在某一时刻，leader 的状态与 follower 的状态很可能不一致，因此它不能处理 leader 挂掉的情况，所以 Zab 协议引入了恢复模式来处理这一问题。从另一角度看，正因为 Zab 的广播过程不需要终止事务，也就是说不需要所有 follower 都返回 ACK 才能进行 COMMIT，而是只需要合法数量（2f+1 台服务器中的 f+1 台） 的follower，也提升了整体的性能。

  


  


作者：两棵橘树

  


链接：https://www.jianshu.com/p/fb527a64deee

  


来源：简书

  


简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。

