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

高可用方法论

下面的表格里，列出了高可用常见的问题和应对措施。



问题	典型案例	增大 MTBF	减小 MTTR

程序、配置 Bug	程序、配置 Bug	提升研发、测试质量，灰度发布	监控告警、快速回滚

机器、机房故障	宕机、机房断电	硬件冗余、多机房	自动故障转移，切流到其他冗余机器、机房

突发流量	上游系统异常重试、外部攻击	上游系统容错调度防雪崩、流量配额、防攻击、防抓取	其他同容量不足

容量不足	主流程容量不足	容量规划、容量预警	限流、降级、熔断弱依赖、快速扩容

依赖服务故障	依赖服务失败率高、超时严重	弱依赖降级解耦，强依赖递归使用前述方法增强可靠性	熔断弱依赖

扩展

扩展是最常见的提升系统可靠性的方法，系统的扩展可以避免单点故障，即一个节点出现了问题造成整个系统无法正常工作。换一个角度讲，一个容易扩展的系统，能够通过扩展来成倍的提升系统能力，轻松应对系统访问量的提升。



一般地，扩展可以分为垂直扩展和水平扩展：



