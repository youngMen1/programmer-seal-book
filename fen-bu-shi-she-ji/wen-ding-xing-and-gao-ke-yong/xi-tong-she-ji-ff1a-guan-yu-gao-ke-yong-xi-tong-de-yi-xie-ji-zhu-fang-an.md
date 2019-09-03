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



