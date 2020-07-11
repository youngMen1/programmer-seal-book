# 1.Apollo配置中心简介（一）
## 1.1.基本介绍
随着程序功能的日益复杂，程序的配置日益增多：各种功能的开关、参数的配置、服务器的地址……对程序配置的期望值也越来越高：配置修改后实时生效，灰度发布，分环境、分集群管理配置，完善的权限、审核机制……在这样的大环境下，传统的通过配置文件、数据库等方式已经越来越无法满足开发人员对配置管理的需求。Apollo配置中心应运而生！
Apollo（阿波罗）是携程框架部门研发的开源配置管理中心，能够集中化管理应用不同环境、不同集群的配置，配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性。

Apollo支持4个维度管理Key-Value格式的配置：

* application (应用)
* environment (环境)
* cluster (集群)
* namespace (命名空间)

同时，Apollo基于开源模式开发，开源地址：`https://github.com/ctripcorp/apollo`

## 1.2.设计架构
![](/static/image/20190808193934865.png)
上图简要描述了Apollo的总体设计，我们可以从下往上看：
* Config Service提供配置的读取、推送等功能，服务对象是Apollo客户端
* Admin Service提供配置的修改、发布等功能，服务对象是Apollo Portal（管理界面）
* Config Service和Admin Service都是多实例、无状态部署，所以需要将自己注册到Eureka中并保持心跳
* 在Eureka之上我们架了一层Meta Server用于封装Eureka的服务发现接口
* Client通过域名访问Meta Server获取Config Service服务列表（IP+Port），而后直接通过IP+Port访问服务，同时在Client侧会做load balance、错误重试
* Portal通过域名访问Meta Server获取Admin Service服务列表（IP+Port），而后直接通过IP+Port访问服务，同时在Portal侧会做load balance、错误重试
* 为了简化部署，我们实际上会把Config Service、Eureka和Meta Server三个逻辑角色部署在同一个JVM进程中

### 服务角色
1.apollo-configservice：提供配置获取接口，提供配置更新推送接口，接口服务对象为Apollo客户端
2.apollo-adminservice：提供配置管理接口，提供配置修改、发布等接口，接口服务对象为Portal，以及Eureka
3.apollo-portal：提供Web界面供用户管理配置
4.apollo-client：Apollo提供的客户端程序，为应用提供配置获取、实时更新等功能

### 配置发布流程
![](/static/image/20190808193607960.png)
上图简要描述了配置发布的大致过程：
* 用户在Portal操作配置发布
* Portal调用Admin Service的接口操作发布
* Admin Service发布配置后，发送ReleaseMessage给各个Config Service
* Config Service收到ReleaseMessage后，通知对应的客户端

### Admin Service 与 Config Service 异步通知流程
Admin Service 在配置发布后，需要通知所有的 Config Service 有配置发布，从而 Config Service 可以通知对应的客户端来拉取最新的配置。从概念上来看，这是一个典型的消息使用场景，Admin Service 作为 producer 发出消息，各个Config Service 作为 consumer 消费消息。通过一个消息组件（Message Queue）就能很好的实现 Admin Service 和 Config Service 的解耦。在实现上，考虑到 Apollo 的实际使用场景，以及为了尽可能减少外部依赖，我们没有采用外部的消息中间件，而是通过数据库实现了一个简单的消息队列。

实现方式：

* 1.Admin Service 在配置发布后会往 ReleaseMessage 表插入一条消息记录，消息内容就是配置发布的 AppId+Cluster+Namespace ，参见 DatabaseMessageSender 。
* 2.Config Service 有一个线程会每秒扫描一次 ReleaseMessage 表，看看是否有新的消息记录，参见 ReleaseMessageScanner 。
* 3.Config Service 如果发现有新的消息记录，那么就会通知到所有的消息监听器（ReleaseMessageListener），如 NotificationControllerV2 ，消息监听器的注册过程参见 ConfigServiceAutoConfiguration 。
* 4.NotificationControllerV2 得到配置发布的 **AppId+Cluster+Namespace** 后，会通知对应的客户端。

示意图：

![](/static/image/20190810173214556.jpg)

### 客户端设计
![](/static/image/20190808195606326.png)

上图简要描述了Apollo客户端的实现原理：

1.客户端和服务端保持了一个长连接，从而能第一时间获得配置更新的推送。
2.客户端还会定时从Apollo配置中心服务端拉取应用的最新配置。
* 这是一个fallback机制，为了防止推送机制失效导致配置不更新
* 客户端定时拉取会上报本地版本，所以一般情况下，对于定时拉取的操作，服务端都会返回304 - Not Modified
* 定时频率默认为每5分钟拉取一次，客户端也可以通过在运行时指定System Property: apollo.refreshInterval来覆盖，单位为分钟。
3.客户端从Apollo配置中心服务端获取到应用的最新配置后，会保存在内存中
4.客户端会把从服务端获取到的配置在本地文件系统缓存一份
* 在遇到服务不可用，或网络不通的时候，依然能从本地恢复配置
应用程序从Apollo客户端获取最新的配置、订阅配置更新通知

