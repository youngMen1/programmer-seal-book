# 1.Spring IOC {#activity-name}

前段时间发布了一个 Spring IOC 相关的文章，[原来 IOC 这么简单](http://mp.weixin.qq.com/s?__biz=Mzg2MjEwMjI1Mg==&mid=2247494025&idx=1&sn=222a31fdfe0206d24049ee9a7dbb5874&chksm=ce0e5e0af979d71cae6433729f2e8d57dd75d612f4606e9a8acebeb9d53858dad4acd3cceb8d&scene=21#wechat_redirect)，反响不错。今天再战 Spring IOC ，根据 Spring 源码写一个带有三级缓存的 IOC。

## 1.1.**Spring 中的 IOC**

Spring 的 IOC 其实很复杂，因为它支持的情况，种类，以及开放的接口，拓展性（如各种PostProcessor）太丰富了。这导致我们在看 Spring 源码的过程中非常吃力，经常点进去一个函数发现很深很深。这篇我主要针对 Spring 的 IOC 中的核心部分，例如 Spring 的 IOC 是如何实现的，Spring 是如何解决循环依赖的这类问题做一个介绍以及一份实现，因为原理是相通的，对于 Spring 对各种情况的逻辑上的处理不做细致的讨论，对原型模式，或是 FactoryBean 类型的 Bean 的不同处理方式不做具体实现。



# 2.参考

再战 Spring IOC：[https://mp.weixin.qq.com/s/kcpPshj3nj3Sd-a-qVJRWg](https://mp.weixin.qq.com/s/kcpPshj3nj3Sd-a-qVJRWg)

卧槽！原来 IOC 这么简单：[https://mp.weixin.qq.com/s/0lfsfxE3mD2rI0fo5E7qkg](https://mp.weixin.qq.com/s/0lfsfxE3mD2rI0fo5E7qkg)

