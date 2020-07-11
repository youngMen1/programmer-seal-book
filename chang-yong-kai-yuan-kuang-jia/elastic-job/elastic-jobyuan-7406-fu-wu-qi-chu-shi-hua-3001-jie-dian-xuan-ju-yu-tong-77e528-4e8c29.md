# 1.Elastic-Job原理--服务器初始化、节点选举与通知(二)
在上一篇博客Elastic-Job原理--简介与示例（一）中我们简单的介绍了一下Elastic-Job提供的功能，这篇博客我们通过分析Elastic-Job的源码，了解学习一下Elastic-Job的初始化、节点选举、配置变更通知等相关的流程。

   Elastic-Job依赖Zookeeper作为注册中心，利用zk的功能完成节点选举、分片和配置变更等相关的功能，接下来我们通过分析源码来了解一下Elastic-Job的节点选举机制。