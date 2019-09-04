# SpringCloud服务注册中心比较:Consul vs Zookeeper vs Etcd vs Eureka

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



