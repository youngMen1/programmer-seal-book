### 分布式锁

* [《分布式锁的几种实现方式》](http://www.hollischuang.com/archives/1716)

  * 基于数据库的分布式锁：优点：操作简单、容易理解。缺点：存在单点问题、数据库性能够开销较大、不可重入；
  * 基于缓存的分布式锁：优点：非阻塞、性能好。缺点：操作不好容易造成锁无法释放的情况。
  * Zookeeper 分布式锁：通过有序临时节点实现锁机制，自己对应的节点需要最小，则被认为是获得了锁。优点：集群可以透明解决单点问题，避免锁不被释放问题，同时锁可以重入。缺点：性能不如缓存方式，吞吐量会随着zk集群规模变大而下降。

* [《基于Zookeeper的分布式锁》](https://www.tuicool.com/articles/VZJr6fY)

  * 清楚的原理描述 + Java 代码示例。

* [《jedisLock—redis分布式锁实现》](https://www.cnblogs.com/0201zcr/p/5942748.html)

  * 基于 setnx\(set if ont exists\)，有则返回false，否则返回true。并支持过期时间。

* [《Memcached 和 Redis 分布式锁方案》](https://blog.csdn.net/albertfly/article/details/77412333)

  * 利用 memcached 的 add（有别于set）操作，当key存在时，返回false。



