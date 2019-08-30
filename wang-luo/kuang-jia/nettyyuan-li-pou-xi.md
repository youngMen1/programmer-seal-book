1. Netty简介

Netty是一个高性能、异步事件驱动的NIO框架，基于JAVA NIO提供的API实现。它提供了对TCP、UDP和文件传输的支持，作为一个异步NIO框架，Netty的所有IO操作都是异步非阻塞的，通过Future-Listener机制，用户可以方便的主动获取或者通过通知机制获得IO操作结果。 作为当前最流行的NIO框架，Netty在互联网领域、大数据分布式计算领域、游戏行业、通信行业等获得了广泛的应用，一些业界著名的开源组件也基于Netty的NIO框架构建。

1. Netty线程模型

在JAVA NIO方面Selector给Reactor模式提供了基础，Netty结合Selector和Reactor模式设计了高效的线程模型。先来看下Reactor模式：

2.1 Reactor模式

Wikipedia这么解释Reactor模型：“The reactor design pattern is an event handling pattern for handling service requests delivered concurrently by one or more inputs. The service handler then demultiplexes the incoming requests and dispatches them synchronously to associated request handlers.”。首先Reactor模式首先是事件驱动的，有一个或者多个并发输入源，有一个Server Handler和多个Request Handlers，这个Service Handler会同步的将输入的请求多路复用的分发给相应的Request Handler。可以如下图所示：
![img](/static/image/20161129103112729.png)





从结构上有点类似生产者和消费者模型，即一个或多个生产者将事件放入一个Queue中，而一个或者多个消费者主动的从这个队列中poll事件来处理；而Reactor模式则没有Queue来做缓冲，每当一个事件输入到Service Handler之后，该Service Handler会主动根据不同的Evnent类型将其分发给对应的Request Handler来处理。



2.2 Reator模式的实现



关于Java NIO 构造Reator模式，Doug lea在《Scalable IO in Java》中给了很好的阐述，这里截取PPT对Reator模式的实现进行说明



1.第一种实现模型如下： 
![img](/static/image/20161129103222584.png)



