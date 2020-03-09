### 1、什么是微服务？

微服务架构是一种架构模式或者说是一种架构风格，它提倡将单一应用程序划分成一组小的服务，每个服务运行在其独立的自己的进程中，服务之间互相协调、互相配合，为用户提供最终价值。

服务之间采用轻量级的通信机制互相沟通（通常是基于HTTP的RESTful API）。每个服务都围绕着具体业务进行构建，并且能够被独立地部署到生产环境、类生产环境等。

另外，应尽量避免统一的、集中式的服务管理机制，对具体的一个服务而言，应根据业务上下文，选择合适的语言、工具对其进行构建，可以有一个非常轻量级的集中式管理来协调这些服务，可以使用不同的语言来编写服务，也可以使用不同的数据存储。

_从技术维度来说_：

微服务化的核心就是将传统的一站式应用，根据业务拆分成一个一个的服务，彻底地去耦合,每一个微服务提供单个业务功能的服务，一个服务做一件事，从技术角度看就是一种小而独立的处理过程，类似进程概念，能够自行单独启动或销毁，拥有自己独立的数据库。

### 2、微服务之间是如何通讯的？

_第一种_：远程过程调用（Remote Procedure Invocation）

直接通过远程过程调用来访问别的service。

示例：REST、gRPC、Apache、Thrift

优点：

```
简单，常见。因为没有中间件代理，系统更简单
```

缺点：

```
只支持请求/响应的模式，不支持别的，比如通知、请求/异步响应、发布/订阅、发布/异步响应  
降低了可用性，因为客户端和服务端在请求过程中必须都是可用的
```

_第二种_：消息

使用异步消息来做服务间通信。服务间通过消息管道来交换消息，从而通信。

示例：Apache Kafka、RabbitMQ

优点:

* 把客户端和服务端解耦，更松耦合 提高可用性，因为消息中间件缓存了消息，直到消费者可以消费

* 支持很多通信机制比如通知、请求/异步响应、发布/订阅、发布/异步响应

缺点:

```
消息中间件有额外的复杂性
```

### 3、springcloud 与dubbo有哪些区别？

相同点：  
    SpringCloud 和Dubbo可以实现RPC远程调用框架，可以实现服务治理。

不同点:  
    SpringCloud是一套目前比较网站微服务框架了，整合了分布式常用解决方案遇到了问题注册中心Eureka、负载均衡器Ribbon ，客户端调用工具Rest和Feign，分布式配置中心Config，服务保护Hystrix，网关Zuul Gateway ，服务链路Zipkin，消息总线Bus等。

Dubbo内部实现功能没有SpringCloud强大（全家桶），只是实现服务治理，缺少分布式配置中心、网关、链路、总线等，如果需要用到这些组件，需要整合其他框架。

**表 Spring Cloud与Dubbo功能对比**

| 功能名称 | Dubbo | Spring Cloud |
| :--- | :--- | :--- |
| 服务注册中心 | ZooKeeper | Spring Cloud Netflix Eureka |
| 服务调用方式 | RPC | REST API |
| 服务网关 | 无 | Spring Cloud Netflix Zuul |
| 断路器 | 不完善 | Spring Cloud Netflix Hystrix |
| 分布式配置 | 无 | Spring Cloud Config |
| 服务跟踪 | 无 | Spring Cloud Sleuth |
| 消息总线 | 无 | Spring Cloud Bus |
| 数据流 | 无 | Spring Cloud Stream |
| 批量任务 | 无 | Spring Cloud Task |

### 4、请谈谈对SpringBoot 和SpringCloud的理解

SpringBoot专注于快速方便的开发单个个体微服务。

* SpringCloud是关注全局的微服务协调整理治理框架，它将SpringBoot开发的一个个单体微服务整合并管理起来，
 
  为各个微服务之间提供，配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选、分布式会话等等集成服务
* SpringBoot可以离开SpringCloud独立使用开发项目，但是SpringCloud离不开SpringBoot，属于依赖的关系.
* SpringBoot专注于快速、方便的开发单个微服务个体，SpringCloud关注全局的服务治理框架。

Spring Boot可以离开Spring Cloud独立使用开发项目，但是Spring Cloud离不开Spring Boot，属于依赖的关系。

### 5、分布式系统面临的问题

### 6、什么是服务熔断，什么是服务降级

### 7、微服务的优缺点分别是什么？说下你在项目开发中碰到的坑？

### 8、你所知道的微服务技术栈有哪些？请列举一二

### 9、什么是 Eureka服务注册与发现

### 10、Eureka的基本架构是什么？

### 11、作为服务注册中心，Eureka比Zookeeper好在哪里?

### 12、什么是 Ribbon负载均衡

### 13、Ribbon负载均衡能干嘛？

### 14、什么是 Feign 负载均衡

### 15、Feign 能干什么

### 16、什么是 Hystrix断路器

### 17、Hystrix断路器能干嘛？

### 18、什么是 zuul路由网关

### 19、什么是SpringCloud Config分布式配置中心

### 20、分布式配置中心能干嘛？



