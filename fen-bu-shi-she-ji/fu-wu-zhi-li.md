## 服务治理

### 服务注册与发现

* [《永不失联！如何实现微服务架构中的服务发现？》](https://blog.csdn.net/jiaolongdy/article/details/51188798)

  * 客户端服务发现模式：客户端直接查询注册表，同时自己负责负载均衡。Eureka 采用这种方式。
  * 服务器端服务发现模式：客户端通过负载均衡查询服务实例。

* [《SpringCloud服务注册中心比较:Consul vs Zookeeper vs Etcd vs Eureka》](https://blog.csdn.net/u010963948/article/details/71730165)

  * CAP支持：Consul（CA）、zookeeper（cp）、etcd（cp） 、euerka（ap）
  * 作者认为目前 Consul 对 Spring cloud 的支持比较好。

* [《基于Zookeeper的服务注册与发现》](http://mobile.51cto.com/news-502394.htm)

  * 优点：API简单、Pinterest，Airbnb 在用、多语言、通过watcher机制来实现配置PUSH，能快速响应配置变化。

### 服务路由控制

* [《分布式服务框架学习笔记4 服务路由》](https://blog.csdn.net/xundh/article/details/59492750)
  * 原则：透明化路由
  * 负载均衡策略：随机、轮询、服务调用延迟、一致性哈希、粘滞连接
  * 本地路由有限策略：injvm\(优先调用jvm内部的服务\)，innative\(优先使用相同物理机的服务\),原则上找距离最近的服务。
  * 配置方式：统一注册表；本地配置；动态下发。



