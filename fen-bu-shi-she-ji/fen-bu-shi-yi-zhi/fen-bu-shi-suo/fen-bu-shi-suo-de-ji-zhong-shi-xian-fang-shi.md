# [分布式锁的几种实现方式](https://www.hollischuang.com/archives/1716)

目前几乎很多大型网站及应用都是分布式部署的，分布式场景中的数据一致性问题一直是一个比较重要的话题。分布式的CAP理论告诉我们“任何一个分布式系统都无法同时满足一致性（Consistency）、可用性（Availability）和分区容错性（Partition tolerance），最多只能同时满足两项。”所以，很多系统在设计之初就要对这三者做出取舍。在互联网领域的绝大多数的场景中，都需要牺牲强一致性来换取系统的高可用性，系统往往只需要保证“最终一致性”，只要这个最终时间是在用户可以接受的范围内即可。

在很多场景中，我们为了保证数据的最终一致性，需要很多的技术方案来支持，比如分布式事务、分布式锁等。有的时候，我们需要保证一个方法在同一时间内只能被同一个线程执行。在单机环境中，Java中其实提供了很多并发处理相关的API，但是这些API在分布式场景中就无能为力了。也就是说单纯的Java Api并不能提供分布式锁的能力。所以针对分布式锁的实现目前有多种方案。

针对分布式锁的实现，目前比较常用的有以下几种方案：

```
基于数据库实现分布式锁 基于缓存（redis，memcached，tair）实现分布式锁 基于Zookeeper实现分布式锁
```

在分析这几种实现方案之前我们先来想一下，我们需要的分布式锁应该是怎么样的？（这里以方法锁为例，资源锁同理）

```
可以保证在分布式部署的应用集群中，同一个方法在同一时间只能被一台机器上的一个线程执行。

这把锁要是一把可重入锁（避免死锁）

这把锁最好是一把阻塞锁（根据业务需求考虑要不要这条）

有高可用的获取锁和释放锁功能

获取锁和释放锁的性能要好
```

### 基于数据库实现分布式锁

#### 基于数据库表

要实现分布式锁，最简单的方式可能就是直接创建一张锁表，然后通过操作该表中的数据来实现了。

当我们要锁住某个方法或资源时，我们就在该表中增加一条记录，想要释放锁的时候就删除这条记录。

创建这样一张数据库表：

    CREATE TABLE `methodLock` (
      `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
      `method_name` varchar(64) NOT NULL DEFAULT '' COMMENT '锁定的方法名',
      `desc` varchar(1024) NOT NULL DEFAULT '备注信息',
      `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '保存数据时间，自动生成',
      PRIMARY KEY (`id`),
      UNIQUE KEY `uidx_method_name` (`method_name `) USING BTREE
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='锁定中的方法';



