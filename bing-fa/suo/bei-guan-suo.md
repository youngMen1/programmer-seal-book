### 悲观锁

悲观锁如果使用不当（锁的条数过多），会引起服务大面积等待。推荐优先使用乐观锁+重试。

* [《【MySQL】悲观锁&乐观锁》](https://www.cnblogs.com/zhiqian-ali/p/6200874.html)

  * 乐观锁的方式：版本号+重试方式
  * 悲观锁：通过 select ... for update 进行行锁\(不可读、不可写，share 锁可读不可写\)。

* [《Mysql查询语句使用select.. for update导致的数据库死锁分析》](https://www.cnblogs.com/Lawson/p/5008741.html)

  * mysql的innodb存储引擎实务锁虽然是锁行，但它内部是锁索引的。
  * 锁相同数据的不同索引条件可能会引起死锁。

* [《Mysql并发时经典常见的死锁原因及解决方法》](https://www.cnblogs.com/zejin2008/p/5262751.html)



