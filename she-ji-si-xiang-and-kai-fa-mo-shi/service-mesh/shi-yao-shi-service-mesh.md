# 什么是Service Mesh？

service mesh是专用的基础设施层，用于使服务之间通信安全，快速和可靠。 如果您正在构建云本机应用程序，则需要服务网格。

在过去的一年中，service mesh已成为云原生栈的关键组件。 Paypal，Ticketmaster和Credit Karma等高流量公司都为其生产应用程序添加了service mesh，今年1月，Linkerd（云原生应用程序的开源service mesh）成为云原生计算基金会的官方项目。 但究竟什么是service mesh呢？ 为什么它突然变得相关？

在本文中，我将定义service mesh，并通过过去十年中应用程序架构的变化来跟踪其发展。 我会把service mesh与API网关，边缘代理和企业服务总线这些看似相关但其实不同的概念区分开来。 最后，我将描述service mesh的发展方向，以及随着云概念的采用而发展的概念。

### 什么是service mesh？

service mesh是用于处理服务与服务之间通信的专用基础设施层。 它负责在包含现代化、云原生应用且拓扑复杂的服务之间可靠地传递请求。 实际上，service mesh通过一系列轻量网络代理来实现，这些代理与应用程序代码一起部署，而且对应用程序不感知。 （但我们会看到，这个想法有所不同。）

service mesh作为单独层的概念与云本机应用程序的兴起有关。 在云原生模型中，单个应用程序可能包含数百个服务; 每个服务可能有数千个实例; 并且每个实例可能处于不断变化的状态，因为它们由像Kubernetes这样的协调器动态调度。 这就导致在应用运行过程中，服务通信不仅非常复杂，而且无处不在。 因此，若要保证端到端的性能和可靠性，对服务通信的管理就变得至关重要。

### service mesh是一个网络模型吗？

service mesh是在TCP/IP之上的一层抽象网络模型。它假设底层L3/L4网络存在并且能够端到端传送字节（它还假设该网络与环境的其他方面一样的不可靠；因此，service mesh也必须能够处理网络故障）。

在某种程度上，service mesh跟TCP/IP类似。 正如TCP堆栈抽象了在网络端点之间可靠地传递字节的机制一样，service mesh抽象了在服务之间可靠地传递请求的机制。 与TCP一样，service mesh不关心实际有效负载或其编码方式。 像TCP一样，service mesh的工作就是在处理任何故障的同时实现一个高级目标（“从A到B发送一些东西”）。

不同的是，除了保证本身的能用性以外，service mesh还提供了一个统一的、应用层次的功能：在应用程序运行过程中引入可视化和控制。servcie mesh的目标很明确，就是把服务通信从不可见、含蓄的架构领域抽离出来，然后转变成整个生态中的一流成员——这样，就可以对其进行监控、管理和控制。

### service mesh能做什么？

在云原生应用程序中可靠地提供请求可能非常复杂。 像Linkerd这样的service mesh通过各种强大的技术来处理这种复杂性：电路中断，延迟感知负载均衡，最终一致（“建议”）服务发现，重试和超时设置。 这些功能必须全部协同工作，这些功能与其运行的复杂环境之间的相互作用可能非常微妙。

举个例子，当通过Linkerd向服务发出请求时，最简单的一个事件时间表如下：

* 1、Linkerd应用动态路由规则来确定请求者想要的服务。 请求是否应该路由到生产环境或staging环境？ 到本地数据中心还是云中的服务？ 是路由到已经测试OK的最新版本，还是生产环境中已经审查锅的较旧的版本？所有这些路由规则都是动态可配置的，并且既可以全局应用，也可以应用于任意部分的流量。
* 2、在找到正确的目的地后，Linkerd从对应的服务发现端点中检索相关的实例池，可能不止一个。 如果这些信息与Linkerd在实践中观察到的信息不同，那么Linkerd会决定要信任哪些信息来源。
* 3、Linkerd根据各种因素选择最有可能返回快速响应的实例，包括观察到的最近请求的延迟。
* 4、Linkerd尝试将请求发送到实例，记录结果的延迟和响应类型。
* 5、如果实例关闭，未响应或无法处理请求，则Linkerd会在另一个实例上重试该请求（但前提是它知道请求是幂等的）。
* 6、如果实例始终返回错误，则Linkerd会将其从负载均衡池中逐出，以便稍后定期重试（例如，实例可能临时故障）。
* 7、如果请求的截止日期已过，则Linkerd会主动使请求失败，而不是通过进一步重试来添加负载。
* 8、Linkerd以度量和分布式跟踪的形式捕获上述行为的各个方面，并将其发送到集中式度量系统。

并且，这仅仅是最简单的情况——Linkerd还可以启动和终止TLS，执行协议升级，动态转换流量以及数据中心之间的故障转移！

需要注意的是，这些功能旨在提供逐点弹性和应用程序范围的弹性。 大规模分布式系统，无论它们如何构建，都有一个明确的特征：小型的、局部的故障很大程度上会升级为系统范围的灾难性故障。 service mesh必须设计成在基础系统接近其极限时通过减少负载和快速失败来防止这些升级。service mesh必须在设计之初就避免这种升级——当底层系统达到极限时，要减少负载和快速失败。

### 为什么service mesh如此重要？

service mesh不是一个新功能，它只是功能所在位置的一个转变。Web应用程序始终必须管理服务通信的复杂性。在过去的十五年中，服务网格模型的起源可以追溯到这些应用程序的演变过程。

考虑2000年代中型Web应用程序的典型架构：三层应用程序。在此模型中，应用程序逻辑，Web服务逻辑和存储逻辑都是单独的层。层之间的通信虽然复杂，但范围有限 - 毕竟只有两跳。没有“网”，但是每次“跳”之间都存在通信逻辑，这些逻辑在每层的代码中进行处理。

当这种架构方法被推到非常高的规模时，它就开始崩溃了。像谷歌，Netflix和Twitter这样的公司面临着巨大的流量需求，实现了有效的云原生方法的前身：应用层被分成许多服务（有时称为“微服务”），层成为拓扑。在这些系统中，广义的通信层突然变得关键，但通常采用“胖客户端”库的形式 - 推特的Finagle，Netflix的Hystrix，以及谷歌的Stubby就是例证。

某种意义上来说，像Finagle，Stubby和Hystrix这样的库可以成为第一代service mesh。虽然它们特定于周围环境的细节，并且需要使用特定的语言和框架，但它们是用于管理服务到服务通信的专用基础架构的形式，并且（在开源Finagle和Hystrix库的情况下）被其他公司使用。

快进到现代云本机应用程序。云原生模型将许多小型服务的微服务方法与两个额外因素相结合：容器（例如Docker），提供资源隔离和依赖管理;以及编排层（例如Kubernetes），它将底层硬件抽象为同质池。

这三个组件允许应用程序适应自然机制，以便在负载下进行扩展，并处理云环境中始终存在的部分故障。但是，随着数百个服务或数千个实例，以及不时重新调度实例的业务流程层，单个请求通过服务拓扑所遵循的路径可能非常复杂，并且由于容器使每个服务都易于编写在另一种语言中，图书馆方法不再可行。

复杂性和关键性的这种组合激发了对与服务器到服务通信的专用层的需求，该专用层与应用程序代码分离并且能够捕获底层环境的高度动态性质。该层是service mesh。

### service mesh的未来

虽然云本地生态系统中的服务网络采用正在迅速增长，但仍有一个广泛而令人兴奋的路线图仍有待探索。 无服务器计算的要求（例如亚马逊的Lambda）直接适用于服务网格的命名和链接模型，并形成其在云原生态系统中的角色的自然扩展。 服务身份和访问策略的角色在云原生环境中仍然非常新生，服务网络很有可能在这里发挥故事的基本部分。 最后，服务网格（如之前的TCP / IP）将继续被推送到底层基础架构中。 正如Linkerd从像Finagle这样的系统发展而来，服务网格作为必须明确添加到云本机堆栈的独立用户空间代理的当前版本也将继续发展。

### 总结

service mesh是云本机堆栈的关键组件。 Linkerd成立仅一年多，它就是Cloud Native Computing Foundation的一部分，拥有一个蓬勃发展的贡献者和用户社区。 收件人的范围从像英国银行业破坏的Monzo这样的创业公司，到Paypal，Ticketmaster和Credit Karma等高规模互联网公司，再到像Houghton Mifflin Harcourt这样经营了数百年的公司。

采用者和贡献者的Linkerd开源社区每天都在展示服务网格模型的价值。 我们致力于打造一个令人惊叹的产品，并继续发展我们令人难以置信的社区。

