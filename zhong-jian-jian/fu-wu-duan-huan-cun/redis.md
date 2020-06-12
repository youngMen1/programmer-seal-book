### Redis

* [《Redis 教程》](http://www.runoob.com/redis/redis-tutorial.html)

* [《redis底层原理》](https://blog.csdn.net/wcf373722432/article/details/78678504)
  * 使用 ziplist 存储链表，ziplist是一种压缩链表，它的好处是更能节省内存空间，因为它所存储的内容都是在连续的内存区域当中的。
  * 使用 skiplist\(跳跃表\)来存储有序集合对象、查找上先从高Level查起、时间复杂度和红黑树相当，实现容易，无锁、并发性好。

* [《Redis持久化方式》](http://doc.redisfans.com/topic/persistence.html)
  * RDB方式：定期备份快照，常用于灾难恢复。优点：通过fork出的进程进行备份，不影响主进程、RDB 在恢复大数据集时的速度比 AOF 的恢复速度要快。缺点：会丢数据。
  * AOF方式：保存操作日志方式。优点：恢复时数据丢失少，缺点：文件大，回复慢。
  * 也可以两者结合使用。

* [《分布式缓存--序列3--原子操作与CAS乐观锁》](https://blog.csdn.net/chunlongyu/article/details/53346436)

#### 架构
* [《Redis单线程架构》](https://blog.csdn.net/sunhuiliang85/article/details/73656830)
#### 回收策略
* [《redis的回收策略》](https://blog.csdn.net/qq_29108585/article/details/63251491)



