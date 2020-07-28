# 1.Dubbo框架设计介绍

## 1.1.基本介绍

Apache Dubbo是一款高性能、轻量级基于Java的RPC开源框架。

文档简短形象的对单一应用架构、垂直应用架构、分布式服务架构、流动计算架构做了一个对比，可以很明白的看出这四个架构所适用的场景，因为业务需求越来越复杂，才会有这一系列的演变。

RPC英文全名为Remote Procedure Call，也叫远程过程调用，其实就是一个计算机通信协议，它是一种通过网络从远程计算机程序上请求服务,而不需要了解底层网络技术的协议。计算机通信协议有很多种，对于开发来说，很多熟悉的是HTTP协议，我这里就做个简单的比较，HTTP协议是属于应用层的，而RPC跨越了传输层和应用层。HTTP本身的三次握手协议，每发送一次请求，都会有一次建立连接的过程，就会带来一定的延迟，并且HTTP本身的报文庞大，而RPC可以按需连接，调用结束后就断掉，也可以是长链接，多个远程过程调用共享同一个链接，可以看出来RPC的效率要高于HTTP，但是相对于开发简单快速的HTTP服务,RPC服务就会显得复杂一些。

回到原先的话题，继续来聊聊dubbo。关于dubbo 的特点分别有连通性、健壮性、伸缩性、以及向未来架构的升级性。特点的详细介绍也可以参考上述链接的官方文档。官方文档拥有的内容我在这就不一一进行阐述了。

因为接下来需要对dubbo各个模块的源码以及原理进行解析，所以介绍一下dubbo的源码库，dubbo框架**2018年2月**已经交由Apache基金会进行孵化，被挂在github开源。

文档地址：`https://dubbo.apache.org/zh-cn/docs/user/quick-start.html`

项目地址：`https://github.com/apache/incubator-dubbo`

## 1.2.整体设计

![](/static/image/12744765-157dd7b682851d26.webp)

官网图例说明：

* 图中左边淡蓝背景的为服务消费方使用的接口，右边淡绿色背景的为服务提供方使用的接口，位于中轴线上的为双方都用到的接口。
* 图中从下至上分为十层，各层均为单向依赖，右边的黑色箭头代表层之间的依赖关系，每一层都可以剥离上层被复用，其中，Service 和 Config 层为 API，其它各层均为 SPI。
* 图中绿色小块的为扩展接口，蓝色小块为实现类，图中只显示用于关联各层的实现类。
* 图中蓝色虚线为初始化过程，即启动时组装链，红色实线为方法调用过程，即运行时调时链，紫色三角箭头为继承，可以把子类看作父类的同一个节点，线上的文字为调用的方法。

### 1.2.1.各层\(10层架构\)说明

* **service 服务层：**这一层是用户自己编写的，不管是服务的接口还是服务的实现。从整体设计图可以看到，接口是提供方和消费方共同使用的，而实现则只在提供方，并不暴露给消费方。这也与我们平时的使用认知相符，服务提供方将接口单独打成一个jar包让消费方使用。

* **config 配置层：**对外配置接口，以 ServiceConfig, ReferenceConfig 为中心，可以直接初始化配置类，也可以通过 spring 解析配置生成配置类。dubbo-config下面包含了config-api和config-spring两个子模块，分别对应了两种配置生成方式。配置类会包含协议、url等信息，是一个比较重的对象。

* **proxy 服务代理层：**服务接口透明代理，生成服务的客户端 Stub 和服务器端 Skeleton, 以 ServiceProxy 为中心，扩展接口为 ProxyFactory。可以看整体设计图，Invoker其实就是负责调用服务提供方真实服务的一个代理，而Proxy则是在消费方负责调用提供方服务的一个代理。

* **registry 注册中心层：**封装服务地址的注册与发现，以服务 URL 为中心，扩展接口为 RegistryFactory, Registry, RegistryService。

* **cluster 路由层：**封装多个提供者的路由及负载均衡，并桥接注册中心，以 Invoker 为中心，扩展接口为 Cluster, Directory, Router, LoadBalance。相当于将集群封装为单个节点，这样外部就可以像单体调用一样处理，不需要考虑集群的情况\(比如多个提供方的情况\)

* **monitor 监控层：**RPC 调用次数和调用时间监控，以 Statistics 为中心，扩展接口为 MonitorFactory, Monitor, MonitorService。会有一些服务调用信息的统计。

* **protocol 远程调用层：**封装 RPC 调用，以 Invocation, Result 为中心，扩展接口为 Protocol, Invoker, Exporter。对协议的封装，dubbo支持多种协议，默认是dubbo协议。

* **exchange 信息交换层：**封装请求响应模式，同步转异步，以 Request, Response 为中心，扩展接口为 Exchanger, ExchangeChannel, ExchangeClient, ExchangeServer。

* **transport 网络传输层：**抽象 mina 和 netty 为统一接口，以 Message 为中心，扩展接口为 Channel, Transporter, Client, Server, Codec。

* **serialize 数据序列化层：**可复用的一些工具，扩展接口为 Serialization, ObjectInput, ObjectOutput, ThreadPool。dubbo的序列化也支持多种协议。

## 1.3. dubbo框架设计

### dubbo整体的结构

![](/static/image/微信截图_20200715165008.png)

可以看到Dubbo被拆分成很多的Maven项目（右边的我还没有截全）接下来我会介绍左边每个模块的大致作用。

如果看过dubbo官方文档的朋友肯定看到过以下这个图：

![](/static/image/2020402645-5bc97b00874e3_articlex.jpg)  
​从以上这个图我们可以清晰的看到各个模块之间依赖关系,其实以上的图只是展示了关键的模块依赖关系，还有部分模块比如dubbo-bootstrap清理模块等，下面我会对各个模块做个简单的介绍，至少弄明白各个模块的作用。

### dubbo-registry——注册中心模块

官方文档的解释：基于注册中心下发地址的集群方式，以及对各种注册中心的抽象，服务目录框架用于服务的注册和服务事件发布和订阅

我的理解是：dubbo的注册中心实现有Multicast注册中心、Zookeeper注册中心、Redis注册中心、Simple注册中心（具体怎么实现我在后面文章中会介绍），这个模块就是封装了dubbo所支持的注册中心的实现。

看看registry目录结构：

![](/static/image/413017614-5bc97d1e64145_articlex.png)

1.dubbo-registry-api：抽象了注册中心的注册和发现，实现了一些公用的方法，让子类只关注部分关键方法。  
2.以下四个包是分别是四种注册中心实现方法的封装，其中`dubbo-registry-default`就是官方文档里面的Simple注册中心。

### dubbo-cluster——集群模块

官方文档的解释：将多个服务提供方伪装为一个提供方，包括：负载均衡, 容错，路由等，集群的地址列表可以是静态配置的，也可以是由注册中心下发。  
![](/static/image/3641195658-5ad5d471dc7dd_articlex.jpg)  
我的理解：它就是一个解决出错情况采用的策略，这个模块里面封装了多种策略的实现方法，并且也支持自己扩展集群容错策略，cluster把多个Invoker伪装成一个Invoker，并且在伪装过程中加入了容错逻辑，失败了，重试下一个。

看看cluster的目录结构：

![](/static/image/3556511465-5bc97d7261a8e_articlex.png)

1.configurator包：配置包，dubbo的基本设计原则是采用URL作为配置信息的统一格式，所有拓展点都通过传递URL携带配置信息，这个包就是用来根据统一的配置规则生成配置信息。  
2.directory包：Directory 代表了多个 Invoker，并且它的值会随着注册中心的服务变更推送而变化 。这里介绍一下Invoker，Invoker是Provider的一个调用Service的抽象，Invoker封装了Provider地址以及Service接口信息。  
3.loadbalance包：封装了负载均衡的实现，负责利用负载均衡算法从多个Invoker中选出具体的一个Invoker用于此次的调用，如果调用失败，则需要重新选择。  
4.merger包：封装了合并返回结果，分组聚合到方法，支持多种数据结构类型。  
5.router包：封装了路由规则的实现，路由规则决定了一次dubbo服务调用的目标服务器，路由规则分两种：条件路由规则和脚本路由规则，并且支持可拓展。  
6.support包：封装了各类Invoker和cluster，包括集群容错模式和分组聚合的cluster以及相关的Invoker。

### dubbo-common——公共逻辑模块

官方文档的解释：包括 Util 类和通用模型。

我的理解：这个应该很通俗易懂，工具类就是一些公用的方法，通用模型就是贯穿整个项目的统一格式的模型，比如URL，上述就提到了URL贯穿了整个项目。

看看common的目录：

![](/static/image/CgqCHl8eRfWANQSTAAHowsC6F8s134.png)  
这个类中的包含义我就不一一讲了，具体的介绍会穿插在后续文章中，因为这些都是工具类的一些实现，包的含义也很明显。

### dubbo-config——配置模块

官方文档的解释：是 Dubbo 对外的 API，用户通过 Config 使用Dubbo，隐藏 Dubbo 所有细节。

我的理解：用户都是使用配置来使用dubbo，dubbo也提供了四种配置方式，包括XML配置、属性配置、API配置、注解配置，配置模块就是实现了这四种配置的功能。

Dubbo 对外暴露的配置都是由该模块进行解析的。例如，dubbo-config-api 子模块负责处理 API 方式使用时的相关配置，dubbo-config-spring 子模块负责处理与 Spring 集成使用时的相关配置方式。有了 dubbo-config 模块，用户只需要了解 Dubbo 配置的规则即可，无须了解 Dubbo 内部的细节。
看看config的目录：

![](/static/image/400949415-5bc97e9dab834_articlex.png)

1.dubbo-config-api：实现了API配置和属性配置的功能。  
2.dubbo-config-spring：实现了XML配置和注解配置的功能。

### dubbo-rpc——远程调用模块

官方文档的解释：抽象各种协议，以及动态代理，只包含一对一的调用，不关心集群的管理，一个远程过程调用的抽象，支持负载均衡、容灾和集群功能。

我的理解：远程调用，最主要的肯定是协议，dubbo提供了许许多多的协议实现，不过官方推荐时使用dubbo自己的协议，还给出了一份性能测试报告。

性能测试报告地址：`http://dubbo.apache.org/zh-cn/docs/user/perf-test.html`

这个模块依赖于dubbo-remoting模块，抽象了各类的协议。

看看rpc的目录：

![](/static/image/微信截图_20200715175822.png)

1.dubbo-rpc-api：抽象了动态代理和各类协议，实现一对一的调用  
2.另外的包都是各个协议的实现。

### dubbo-remoting——远程通信模块

官方文档的解释：相当于 Dubbo 协议的实现，如果 RPC 用 RMI协议则不需要使用此包，网络通信框架，实现了 sync-over-async 和 request-response 消息机制。

我的理解：提供了多种客户端和服务端通信功能，比如基于Grizzly、Netty、Tomcat等等，RPC用除了RMI的协议都要用到此模块。

看看remoting的目录：  
![](/static/image/2915119122-5bc97f1442fec_articlex.png)

1.dubbo-remoting-api：定义了客户端和服务端的接口。  
2.dubbo-remoting-grizzly：基于Grizzly实现的Client和Server。  
3.dubbo-remoting-http：基于Jetty或Tomcat实现的Client和Server。  
4.dubbo-remoting-mina：基于Mina实现的Client和Server。  
5.dubbo-remoting-netty：基于Netty3实现的Client和Server。  
6.Dubbo-remoting-netty4：基于Netty4实现的Client和Server。  
7.dubbo-remoting-p2p：P2P服务器，注册中心multicast中会用到这个服务器使用。  
8.dubbo-remoting-zookeeper：封装了Zookeeper Client ，和 Zookeeper Server 通信。

### dubbo-container——容器模块

官方文档的解释：是一个 Standlone 的容器，以简单的 Main 加载 Spring 启动，因为服务通常不需要 Tomcat/JBoss 等 Web 容器的特性，没必要用 Web 容器去加载服务。

我的理解：因为后台服务不需要Tomcat/JBoss 等 Web 容器的功能，不需要用这些厚实的容器去加载服务提供方，既资源浪费，又增加复杂度。服务容器只是一个简单的Main方法，加载一些内置的容器，也支持扩展容器。

看看container的目录：  
![](/static/image/486980947-5bc97f99a9383_articlex.png)  
dubbo-container-api：定义了Container接口，实现了服务加载的Main方法。  
其他三个分别提供了对应的容器，供Main方法加载。

### dubbo-monitor——监控模块

官方文档的解释：统计服务调用次数，调用时间的，调用链跟踪的服务,统计服务的调用次调和调用时间的日志服务称之为“服务监控中心”

我的理解：这个模块很清楚，就是对服务的监控。

看看monitor的目录：

![](/static/image/4259709195-5bc97fbf1e7fe_articlex.png)

1.dubbo-monitor-api：定义了monitor相关的接口，实现了监控所需要的过滤器。  
2.dubbo-monitor-default：实现了dubbo监控相关的功能。

### dubbo-bootstrap——清理模块

这个模块只有一个类，是作为dubbo的引导类，并且在停止期间进行清理资源。具体的介绍我在后续文章中讲解。

### dubbo-demo——示例模块

这个模块是快速启动示例，其中包含了服务提供方和调用方，注册中心用的是multicast，用XML配置方法，具体的介绍可以看官方文档。

![](/static/image/微信截图_20200715175637.png)

示例介绍地址：`http://dubbo.apache.org/zh-cn/docs/user/quick-start.html`

### dubbo-filter——过滤器模块

这个模块提供了内置的一些过滤器。

看看filter的目录：  
![](/static/image/1004755792-5bc980382351a_articlex.png)

1.dubbo-filter-cache：提供缓存过滤器。  
2.dubbo-filter-validation：提供参数验证过滤器。

### dubbo-plugin——插件模块

该模块提供了内置的插件。

看看plugin的目录：

![](/static/image/637508015-5bc98051a2650_articlex.png)

1.dubbo-qos：提供了在线运维的命令。

### dubbo-serialization——序列化模块

该模块中封装了各类序列化框架的支持实现。

看看serialization的目录：

![](/static/image/微信截图_20200715180236.png)

dubbo-serialization-api：定义了Serialization的接口以及数据输入输出的接口。  
其他的包都是实现了对应的序列化框架的方法。dubbo内置的就是这几类的序列化框架，序列化也支持扩展。

### dubbo-test——测试模块

这个模块封装了针对dubbo的性能测试、兼容性测试等功能。

看看test的目录：

![](/static/image/3683560091-5bc980997aea1_articlex.png)

1.dubbo-test-benchmark：对性能的测试。  
2.dubbo-test-compatibility：对兼容性的测试，对spring3对兼容性测试。  
3.dubbo-test-examples：测试所使用的示例。  
4.dubbo-test-integration：测试所需的pom文件

### 下面我来讲讲dubbo中Maven相关的pom文件

1.`dubbo-bom/pom.xml`，利用Maven BOM统一定义了dubbo的版本号。`dubbo-test`和`dubbo-demo`的pom文件中都会引用`dubbo-bom/pom.xml`，以`dubbo-demo`都pom举例子：  
![](/static/image/2905388040-5bc980ec7d454_articlex.png)

2.`dubbo-dependencies-bom/pom.xml`：利用Maven BOM统一定义了dubbo依赖的第三方库的版本号。`dubbo-parent`会引入该bom：

![](/static/image/2377277672-5bc9818cd17fe_articlex.png)

3.`all/pow.xml`：定义了dubbo的打包脚本，使用dubbo库的时候，需要引入改pom文件。  
4.`dubbo-parent`：是dubbo的父pom，dubbo的maven模块都会引入该pom文件。

# 2.参考

Dubbo源码解析：[https://segmentfault.com/a/1190000016842868](https://segmentfault.com/a/1190000016842868)

