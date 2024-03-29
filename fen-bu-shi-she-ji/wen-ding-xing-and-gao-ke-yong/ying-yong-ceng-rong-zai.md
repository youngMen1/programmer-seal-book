### 应用层容灾

* [《防雪崩利器：熔断器 Hystrix 的原理与使用》](https://segmentfault.com/a/1190000005988895)

  * 雪崩效应原因：硬件故障、硬件故障、程序Bug、重试加大流量、用户大量请求。
  * 雪崩的对策：限流、改进缓存模式\(缓存预加载、同步调用改异步\)、自动扩容、降级。
  * Hystrix设计原则：
    * 资源隔离：Hystrix通过将每个依赖服务分配独立的线程池进行资源隔离, 从而避免服务雪崩。
    * 熔断开关：服务的健康状况 = 请求失败数 / 请求总数，通过阈值设定和滑动窗口控制开关。
    * 命令模式：通过继承 HystrixCommand 来包装服务调用逻辑。

* [《缓存穿透，缓存击穿，缓存雪崩解决方案分析》](https://blog.csdn.net/zeb_perfect/article/details/54135506)

* [《缓存击穿、失效以及热点key问题》](https://blog.csdn.net/zeb_perfect/article/details/54135506)

  * 主要策略：失效瞬间：单机使用锁；使用分布式锁；不过期；
  * 热点数据：热点数据单独存储；使用本地缓存；分成多个子key；



