# 24 | 业务代码写完，就意味着生产就绪了？

你好，我是朱晔。

今天，我们来聊聊业务代码写完，是不是就意味着生产就绪，可以直接投产了。所谓生产就绪（Production-ready），是指应用开发完成要投入生产环境，开发层面需要额外做的一些工作。

在我看来，如果应用只是开发完成了功能代码，然后就直接投产，那意味着应用其实在裸奔。在这种情况下，遇到问题因为缺乏有效的监控导致无法排查定位问题，同时很可能遇到问题我们自己都不知道，需要依靠用户反馈才知道应用出了问题。

那么，生产就绪需要做哪些工作呢？我认为，以下三方面的工作最重要。

**第一，提供健康检测接口。**传统采用 ping 的方式对应用进行探活检测并不准确。有的时候，应用的关键内部或外部依赖已经离线，导致其根本无法正常工作，但其对外的 Web 端口或管理端口是可以 ping 通的。我们应该提供一个专有的监控检测接口，并尽可能触达一些内部组件。

**第二，暴露应用内部信息。**应用内部诸如线程池、内存队列等组件，往往在应用内部扮演了重要的角色，如果应用或应用框架可以对外暴露这些重要信息，并加以监控，那么就有可能在诸如 OOM 等重大问题暴露之前发现蛛丝马迹，避免出现更大的问题。

**第三，建立应用指标 Metrics 监控。**Metrics 可以翻译为度量或者指标，指的是对于一些关键信息以可聚合的、数值的形式做定期统计，并绘制出各种趋势图表。这里的指标监控，包括两个方面：一是，应用内部重要组件的指标监控，比如 JVM 的一些指标、接口的 QPS 等；二是，应用的业务数据的监控，比如电商订单量、游戏在线人数等。

今天，我就通过实际案例，和你聊聊如何快速实现这三方面的工作。

## 准备工作：配置 Spring Boot Actuator

Spring Boot 有一个 Actuator 模块，封装了诸如健康检测、应用内部信息、Metrics 指标等生产就绪的功能。今天这一讲后面的内容都是基于 Actuator 的，因此我们需要先完成 Actuator 的引入和配置。

我们可以像这样在 pom 中通过添加依赖的方式引入 Actuator：


```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
之后，你就可以直接使用 Actuator 了，但还要注意一些重要的配置：


* 如果你不希望 Web 应用的 Actuator 管理端口和应用端口重合的话，可以使用 management.server.port 设置独立的端口。

* Actuator 自带了很多开箱即用提供信息的端点（Endpoint），可以通过 JMX 或 Web 两种方式进行暴露。考虑到有些信息比较敏感，这些内置的端点默认不是完全开启的，你可以通过官网查看这些默认值。在这里，为了方便后续 Demo，我们设置所有端点通过 Web 方式开启。

* 默认情况下，Actuator 的 Web 访问方式的根地址为 /actuator，可以通过 management.endpoints.web.base-path 参数进行修改。我来演示下，如何将其修改为 /admin。


```
https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-endpoints-exposing-endpoints
```


```

management.server.port=45679
management.endpoints.web.exposure.include=*
management.endpoints.web.base-path=/admin
```

现在，你就可以访问 http://localhost:45679/admin ，来查看 Actuator 的所有功能 URL 了：
420d5b3d9c10934e380e555c2347834b.png

其中，大部分端点提供的是只读信息，比如查询 Spring 的 Bean、ConfigurableEnvironment、定时任务、SpringBoot 自动配置、Spring MVC 映射等；少部分端点还提供了修改功能，比如优雅关闭程序、下载线程 Dump、下载堆 Dump、修改日志级别等。


你可以访问这里，查看所有这些端点的功能，详细了解它们提供的信息以及实现的操作。此外，我再分享一个不错的 Spring Boot 管理工具Spring Boot Admin，它把大部分 Actuator 端点提供的功能封装为了 Web UI。


```
https://docs.spring.io/spring-boot/docs/2.2.4.RELEASE/actuator-api//html/

https://github.com/codecentric/spring-boot-admin
```

## 健康检测需要触达关键组件

在这一讲开始我们提到，健康检测接口可以让监控系统或发布工具知晓应用的真实健康状态，比 ping 应用端口更可靠。不过，要达到这种效果最关键的是，我们能确保健康检测接口可以探查到关键组件的状态。

好在 Spring Boot Actuator 帮我们预先实现了诸如数据库、InfluxDB、Elasticsearch、Redis、RabbitMQ 等三方系统的健康检测指示器 HealthIndicator。

通过 Spring Boot 的自动配置，这些指示器会自动生效。当这些组件有问题的时候，HealthIndicator 会返回 DOWN 或 OUT_OF_SERVICE 状态，health 端点 HTTP 响应状态码也会变为 503，我们可以以此来配置程序健康状态监控报警。

为了演示，我们可以修改配置文件，把 management.endpoint.health.show-details 参数设置为 always，让所有用户都可以直接查看各个组件的健康情况（如果配置为 when-authorized，那么可以结合 management.endpoint.health.roles 配置授权的角色）：



```

management.endpoint.health.show-details=always
```

访问 health 端点可以看到，数据库、磁盘、RabbitMQ、Redis 等组件健康状态是 UP，整个应用的状态也是 UP：
3c98443ebb76b65c4231aa35086dc8be.png
在了解了基本配置之后，我们考虑一下，如果程序依赖一个很重要的三方服务，我们希望这个服务无法访问的时候，应用本身的健康状态也是 DOWN。

比如三方服务有一个 user 接口，出现异常的概率是 50%：



















