# 1.分布式RPC框架性能大比拼 dubbo、motan、rpcx、gRPC、thrift的性能比较

[Dubbo](http://dubbo.io/) 是阿里巴巴公司开源的一个Java高性能优秀的服务框架，使得应用可通过高性能的 RPC 实现服务的输出和输入功能，可以和 Spring框架无缝集成。不过，略有遗憾的是，据说在淘宝内部，dubbo由于跟淘宝另一个类似的框架HSF（非开源）有竞争关系，导致dubbo团队已经解散（参见[http://www.oschina.net/news/55059/druid-1-0-9](http://www.oschina.net/news/55059/druid-1-0-9) 中的评论），反到是当当网的扩展版本仍在持续发展，墙内开花墙外香。其它的一些知名电商如当当、京东、国美维护了自己的分支或者在dubbo的基础开发，但是官方的库缺乏维护，相关的依赖类比如Spring，Netty还是很老的版本\(Spring 3.2.16.RELEASE, netty 3.2.5.Final\),倒是有些网友写了升级Spring和Netty的插件。

[Motan](https://github.com/weibocom/motan)是新浪微博开源的一个Java 框架。它诞生的比较晚，起于2013年，2016年5月开源。Motan 在微博平台中已经广泛应用，每天为数百个服务完成近千亿次的调用。

[rpcx](https://github.com/smallnest/rpcx)是Go语言生态圈的Dubbo， 比Dubbo更轻量，实现了Dubbo的许多特性，借助于Go语言优秀的并发特性和简洁语法，可以使用较少的代码实现分布式的RPC服务。

[gRPC](http://www.grpc.io/)是Google开发的高性能、通用的开源RPC框架，其由Google主要面向移动应用开发并基于HTTP/2协议标准而设计，基于ProtoBuf\(Protocol Buffers\)序列化协议开发，且支持众多开发语言。本身它不是分布式的，所以要实现上面的框架的功能需要进一步的开发。

[thrift](https://thrift.apache.org/)是Apache的一个跨语言的高性能的服务框架，也得到了广泛的应用。

以下是它们的功能比较：

