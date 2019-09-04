# SpringCloud服务注册中心比较:Consul vs Zookeeper vs Etcd vs Eureka

分布式系统的CAP理论：理论首先把分布式系统中的三个特性进行了如下归纳：

一致性（C）：在分布式系统中的所有数据备份，在同一时刻是否同样的值。（等同于所有节点访问同一份最新的数据副本）  
可用性（A）：在集群中一部分节点故障后，集群整体是否还能响应客户端的读写请求。（对数据更新具备高可用性）  
分区容错性（P）：以实际效果而言，分区相当于对通信的时限要求。系统如果不能在时限内达成数据一致性，就意味着发生了分区的情况，必须就当前操作在C和A之间做出选择。



这里就平时经常用到的服务发现的产品进行下特性的对比，首先看下结论:

| Feature | Consul | zookeeper | etcd | euerka |
| :--- | :--- | :--- | :--- | :--- |
| 服务健康检查 | 服务状态，内存，硬盘等 | \(弱\)长连接，keepalive | 连接心跳 | 可配支持 |
| 多数据中心 | 支持 | — | — | — |
| kv存储服务 | 支持 | 支持 | 支持 | — |
| 一致性 | raft | paxos | raft | — |
| cap | ca | cp | cp | ap |
| 使用接口\(多语言能力\) | 支持http和dns | 客户端 | http/grpc | http（sidecar） |
| watch支持 | 全量/支持long polling | 支持 | 支持 long polling | 支持 long polling/大部分增量 |
| 自身监控 | metrics | — | metrics | metrics |
| 安全 | acl /https | acl | https支持（弱） | — |
| spring cloud集成 | 已支持 | 已支持 | 已支持 | 已支持 |

服务的健康检查

Euraka 使用时需要显式配置健康检查支持；Zookeeper,Etcd 则在失去了和服务进程的连接情况下任务不健康，而 Consul 相对更为详细点，比如内存是否已使用了90%，文件系统的空间是不是快不足了。

* 多数据中心支持

Consul 通过 WAN 的 Gossip 协议，完成跨数据中心的同步；而且其他的产品则需要额外的开发工作来实现；

* KV 存储服务

除了 Eureka ,其他几款都能够对外支持 k-v 的存储服务，所以后面会讲到这几款产品追求高一致性的重要原因。而提供存储服务，也能够较好的转化为动态配置服务哦。

* 产品设计中 CAP 理论的取舍

Eureka 典型的 AP,作为分布式场景下的服务发现的产品较为合适，服务发现场景的可用性优先级较高，一致性并不是特别致命。其次 CA 类型的场景 Consul,也能提供较高的可用性，并能 k-v store 服务保证一致性。 而Zookeeper,Etcd则是CP类型 牺牲可用性，在服务发现场景并没太大优势；

* 多语言能力与对外提供服务的接入协议

Zookeeper的跨语言支持较弱，其他几款支持 http11 提供接入的可能。Euraka 一般通过 sidecar的方式提供多语言客户端的接入支持。Etcd 还提供了Grpc的支持。 Consul除了标准的Rest服务api,还提供了DNS的支持。

* Watch的支持（客户端观察到服务提供者变化）

Zookeeper 支持服务器端推送变化，Eureka 2.0\(正在开发中\)也计划支持。 Eureka 1,Consul,Etcd则都通过长轮询的方式来实现变化的感知；

* 自身集群的监控

除了 Zookeeper ,其他几款都默认支持 metrics，运维者可以搜集并报警这些度量信息达到监控目的；

* 安全

Consul,Zookeeper 支持ACL，另外 Consul,Etcd 支持安全通道https.

* Spring Cloud的集成

目前都有相对应的 boot starter，提供了集成能力。

总的来看，目前Consul 自身功能，和 spring cloud 对其集成的支持都相对较为完善，而且运维的复杂度较为简单（没有详细列出讨论），Eureka 设计上比较符合场景，但还需持续的完善。

