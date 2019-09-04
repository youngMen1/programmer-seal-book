## **为什么使用服务发现？**

想象一下，如果你在写代码调用一个有REST API或Thrift API的服务，你的代码需要知道一个服务实例的网络地址（IP地址和端口）。运行在物理硬件上的传统应用中，服务实例的网络地址是相对静态的，你的代码可以从一个很少更新的配置文件中读取网络地址。

在一个现代的，基于云的微服务应用中，这个问题就变得复杂多了，如下图所示：

![img](/static/image/640.webp)

服务实例的网络地址是动态分配的。而且，由于自动扩展，失败和更新，服务实例的配置也经常变化。这样一来，你的客户端代码需要一套更精细的服务发现机制。

有两种主要的服务发现模式：客户端服务发现（client-side discovery）和服务器端服务发现（server-side discovery）。我们首先来看下客户端服务发现。

## **客户端服务发现模式**

当使用客户端服务发现的时候，客户端负责决定可用的服务实例的网络地址，以及围绕他们的负载均衡。客户端向服务注册表（service registry）发送一个请求，服务注册表是一个可用服务实例的数据库。客户端使用一个负载均衡算法，去选择一个可用的服务实例，来响应这个请求，下图展示了这种模式的架构：

641.webp

一个服务实例被启动时，它的网络地址会被写到注册表上；当服务实例终止时，再从注册表中删除。这个服务实例的注册表通过心跳机制动态刷新。

Netflix OSS提供了一个客户端服务发现的好例子。Netflix Eureka是一个服务注册表，提供了REST API用来管理服务实例的注册和查询可用的实例。Netflix Ribbon是一个IPC客户端，和Eureka一起处理可用服务实例的负载均衡。下面会深入讨论Eureka。

**客户端的服务发现模式有优势也有缺点。**这种模式相对直接，但是除了服务注册表，没有其它动态的部分了。而且，由于客户端知道可用的服务实例，可以做到智能的，应用明确的负载均衡决策，比如一直用hash算法。这种模式的一个重大缺陷在于，客户端和服务注册表是一一对应的，必须为服务客户端用到的每一种编程语言和框架实现客户端服务发现逻辑。

## **服务器端服务发现模式**

下图展示了这种模式的架构

642.webp

客户端通过负载均衡器向一个服务发送请求，这个负载均衡器会查询服务注册表，并将请求路由到可用的服务实例上。通过客户端的服务发现，服务实例在服务注册表上被注册和注销。

  


AWS的ELB（Elastic Load Blancer）就是一个服务器端服务发现路由器。一个ELB通常被用来均衡来自互联网的外部流量，也可以用ELB去均衡流向VPC（Virtual Private Cloud）的流量。一个客户端通过ELB发送请求（HTTP或TCP）时，使用的是DNS，ELB会均衡这些注册的EC2实例或ECS（EC2 Container Service）容器的流量。没有另外的服务注册表，EC2实例和ECS容器也只会在ELB上注册。

  


HTTP服务器和类似Nginx、Nginx Plus的负载均衡器也可以被用做服务器端服务发现负载均衡器。例如，Consul Template可以用来动态配置Nginx的反向代理。

  


Consul Template定期从存储在Consul服务注册表的数据中，生成任意的配置文件。每当文件变化时，会运行一个shell命令。比如，Consul Template可以生成一个配置反向代理的nginx.conf文件，然后运行一个命令告诉Nginx去重新加载配置。还有一个更复杂的实现，通过HTTP API或DNS去动态地重新配置Nginx Plus。

  


有些部署环境，比如Kubernetes和Marathon会在集群中的每个host上运行一个代理。这个代理承担了服务器端服务发现负载均衡器的角色。为了向一个服务发送一个请求，一个客户端使用host的IP地址和服务分配的端口，通过代理路由这个请求。这个代理会直接将请求发送到集群上可用的服务实例。

  


服务器端服务发现模式也是优势和缺陷并存。最大的好处在于服务发现的细节被从客户端中抽象出来，客户端只需要向负载均衡器发送请求，不需要为服务客户端使用的每一种语言和框架，实现服务发现逻辑；另外，这种模式也有一些问题，除非这个负载均衡器是由部署环境提供的，又是另一个高需要启动和管理的可用的系统组件。

  


![](http://mmbiz.qpic.cn/mmbiz/CiaJxoTn5FwrPtaEibEr6eHvJVuVhviaTic7xdW7Dzu3qaWKxVyLZnicBr2yLeoicuxiapt8TtQ8oKQdbn0iaFNyaygjmg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**服务注册表（Service Registry）**

![](http://mmbiz.qpic.cn/mmbiz/CiaJxoTn5FwrPtaEibEr6eHvJVuVhviaTic7AYMJNqNJSt27sMmIUib7KUCOib0ltrwVOoeJFDqybygPz08EbU1dwZpw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

服务注册表是服务发现的关键部分，是一个包含了服务实例的网络地址的数据库，必须是高可用和最新的。客户端可以缓存从服务注册表处获得的网络地址。但是，这些信息最终会失效，客户端会找不到服务实例。所以，服务注册表由一个服务器集群组成，通过应用协议来保持一致性。

  


正如上面提到的，Netflix Eureka是一个服务注册表的好例子。它提供了一个REST API用来注册和查询服务实例。一个服务实例通过POST请求来注册自己的网络位置，每隔30秒要通过一个PUT请求重新注册。注册表中的一个条目会因为一个HTTP DELETE请求或实例注册超时而被删除，客户端通过一个HTTP GET请求来检索注册的服务实例。

  


Netflix通过在每个EC2的可用区中，运行一个或多个Eureka服务器实现高可用。每个运行在EC2实例上的Eureka服务器都有一个弹性的IP地址。DNS TEXT records用来存储Eureka集群配置，实际上是从可用区到Eureka服务器网络地址的列表的映射。当一个Eureka服务器启动时，会向DNS发送请求，检索Eureka集群的配置，定位节点，并为自己分配一个未占用的弹性IP地址。

  


Eureka客户端（服务和服务客户端）查询DNS去寻找Eureka服务器的网络地址。客户端更想使用这个可用区内的Eureka服务器，如果没有可用的Eureka服务器，客户端会用另一个可用区内的Eureka服务器。

  


其它服务注册的例子包括：

  


* Etcd：一个高可用，分布式，一致的key-value存储，用来共享配置和服务发现。Kubernetes和Cloudfoundry都使用了etcd；

* Consul：一个发现和配置服务的工具。客户端可以利用它提供的API，注册和发现服务。Consul可以执行监控检测来实现服务的高可用；

* Apache Zookeeper：一个常用的，为分布式应用设计的高可用协调服务，最开始Zookeeper是Hadoop的子项目，现在已经顶级项目了。

  


一些系统，比如Kubernetes，Marathon和AWS没有一个明确的服务注册组件，这项功能是内置在基础设置中的。

  


下面我们来看看服务实例如何在注册表中注册。

  


![](http://mmbiz.qpic.cn/mmbiz/CiaJxoTn5FwrPtaEibEr6eHvJVuVhviaTic7xdW7Dzu3qaWKxVyLZnicBr2yLeoicuxiapt8TtQ8oKQdbn0iaFNyaygjmg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**服务注册（Service Registration）**

![](http://mmbiz.qpic.cn/mmbiz/CiaJxoTn5FwrPtaEibEr6eHvJVuVhviaTic7AYMJNqNJSt27sMmIUib7KUCOib0ltrwVOoeJFDqybygPz08EbU1dwZpw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

前面提到了，服务实例必须要从注册表中注册和注销，有很多种方式来处理注册和注销的过程。一个选择是服务实例自己注册，即self-registration模式。另一种选择是其它的系统组件管理服务实例的注册，即第third-party registration模式。

  


![](http://mmbiz.qpic.cn/mmbiz/CiaJxoTn5FwrPtaEibEr6eHvJVuVhviaTic7xdW7Dzu3qaWKxVyLZnicBr2yLeoicuxiapt8TtQ8oKQdbn0iaFNyaygjmg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**The Self-Registration Pattern**

![](http://mmbiz.qpic.cn/mmbiz/CiaJxoTn5FwrPtaEibEr6eHvJVuVhviaTic7AYMJNqNJSt27sMmIUib7KUCOib0ltrwVOoeJFDqybygPz08EbU1dwZpw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在self-registration模式中，服务实例负责从服务注册表中注册和注销。如果需要的话，一个服务实例发送心跳请求防止注册过期。下图展示了这种模式的架构：644.webp

