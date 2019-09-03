# 系统设计：关于高可用系统的一些技术方案

### 文章目录

* [系统设计：关于高可用系统的一些技术方案](https://blog.csdn.net/hustspy1990/article/details/78008324#_1)
* * [高可用方法论](https://blog.csdn.net/hustspy1990/article/details/78008324#_5)
  * [扩展](https://blog.csdn.net/hustspy1990/article/details/78008324#_17)
  * [隔离](https://blog.csdn.net/hustspy1990/article/details/78008324#_37)
  * [解耦](https://blog.csdn.net/hustspy1990/article/details/78008324#_47)
  * [限流](https://blog.csdn.net/hustspy1990/article/details/78008324#_58)
  * * [分类](https://blog.csdn.net/hustspy1990/article/details/78008324#_65)
    * [漏桶算法](https://blog.csdn.net/hustspy1990/article/details/78008324#_72)
    * [令牌桶算法](https://blog.csdn.net/hustspy1990/article/details/78008324#_80)
    * [滑动窗口计数法](https://blog.csdn.net/hustspy1990/article/details/78008324#_88)
    * [动态限流](https://blog.csdn.net/hustspy1990/article/details/78008324#_97)
  * [降级](https://blog.csdn.net/hustspy1990/article/details/78008324#_104)
  * [熔断](https://blog.csdn.net/hustspy1990/article/details/78008324#_113)
  * [发布相关](https://blog.csdn.net/hustspy1990/article/details/78008324#_127)
  * * [模块级自动化测试](https://blog.csdn.net/hustspy1990/article/details/78008324#_129)
    * [灰度发布 & 回滚](https://blog.csdn.net/hustspy1990/article/details/78008324#___146)
  * [故障演练](https://blog.csdn.net/hustspy1990/article/details/78008324#_155)
  * [自动化运维-故障自愈](https://blog.csdn.net/hustspy1990/article/details/78008324#_161)
  * [事件系统](https://blog.csdn.net/hustspy1990/article/details/78008324#_165)
  * [其他](https://blog.csdn.net/hustspy1990/article/details/78008324#_171)
  * [总结](https://blog.csdn.net/hustspy1990/article/details/78008324#_176)
  * [参考资料](https://blog.csdn.net/hustspy1990/article/details/78008324#_196)

# 系统设计：关于高可用系统的一些技术方案

可靠的系统是业务稳定、快速发展的基石。那么，如何做到系统高可靠、高可用呢？下面首先讲一下高可用需要面临的常见问题，再从技术方面介绍几种提高系统可靠性、可用性的方法。

## 高可用方法论

下面的表格里，列出了高可用常见的问题和应对措施。

![img](/static/image/微信截图_20190903161439.png)扩展

扩展是最常见的提升系统可靠性的方法，系统的扩展可以避免单点故障，即一个节点出现了问题造成整个系统无法正常工作。换一个角度讲，一个容易扩展的系统，能够通过扩展来成倍的提升系统能力，轻松应对系统访问量的提升。

一般地，扩展可以分为垂直扩展和水平扩展：

垂直扩展：是在同一逻辑单元里添加资源从而满足系统处理能力上升的需求。比如，当机器内存不够时，我们可以帮机器增加内存，或者数据存不下时，我们为机器挂载新的磁盘。

垂直扩展能够提升系统处理能力，但不能解决单点故障问题。

优点：扩展简单。

缺点：扩展能力有限。

水平扩展：通过增加一个或多个逻辑单元，并使得它们像整体一样的工作。

水平扩展，通过冗余部署解决了单点故障，同时又提升了系统处理能力。

优点：扩展能力强。

缺点：增加系统复杂度，维护成本高，系统需要是无状态的、可分布式的。

可扩展性系数 scalability factor 通常用来衡量一个系统的扩展能力，当增加 1 单元的资源时，系统处理能力只增加了 0.95 单元，那么可扩展性系数就是 95%。当系统在持续的扩展中，可扩展系数始终保持不变，我们就称这种扩展是线性可扩展。

在实际应用中，水平扩展最常见：

通常我们在部署应用服务器的时候，都会部署多台，然后使用 nginx 来做负载均衡，nginx 使用心跳机制来检测服务器的正常与否，无响应的服务就从集群中剔除。这样的集群中每台服务器的角色是相同的，同时提供一样的服务。

在数据库的部署中，为了防止单点故障，一般会使用一主多从，通常写操作只发生在主库。不同数据库之间角色不同。当主机宕机时，一台从库可以自动切换为主机提供服务。

隔离

隔离，是对什么进行隔离呢？是对系统、业务所占有的资源进行隔离，限制某个业务对资源的占用数量，避免一个业务占用整个系统资源，对其他业务造成影响。



隔离级别按粒度从小到大，可以分为线程池隔离、进程隔离、模块隔离、应用隔离、机房隔离。在数据库的使用中，还经常用到读写分离。



线程池隔离：不同的业务使用不同的线程池，避免低优先级的任务阻塞高优先级的任务。或者高优先级的任务过多，导致低优先级任务永远不会执行。

进程隔离：Linux 中有用于进程资源隔离的 Linux CGroup，通过物理限制的方式为进程间资源控制提供了简单的实现方式，为 Linux Container 技术、虚拟化技术的发展奠定了技术基础。在工作中的实际应用，可以看看这篇文章：日志压缩资源消耗优化: Linux CGroup 的使用。

模块隔离、应用隔离：很多线上故障的发生源于代码修改后，测试不到位导致。按照代码或业务的易变程度来划分模块或应用，把变化较少的划分到一个模块或应用中，变化较多的划分到另一个模块或应用中。减少代码修改影响的范围，也就减少了测试的工作量，减少了故障出现的概率。

机房隔离：主要是为了避免单个机房网络问题或断电吧。

读写分离：一方面，将对实时性要求不高的读操作，放到 DB 从库上执行，有利于减轻 DB 主库的压力。另一方面，将一些耗时离线业务 sql 放到 DB 从库上执行，能够减少慢 sql 对 DB 主库的影响，保证线上业务的稳定可靠。

————————————————

版权声明：本文为CSDN博主「albon\_arith」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/hustspy1990/article/details/78008324

