### 原理

* [《MySQL的InnoDB索引原理详解》](http://www.admin10000.com/document/5372.html)

* [《MySQL存储引擎－－MyISAM与InnoDB区别》](https://blog.csdn.net/xifeijian/article/details/20316775)

  * 两种类型最主要的差别就是Innodb 支持事务处理与外键和行级锁

* [《myisam和innodb索引实现的不同》](https://www.2cto.com/database/201211/172380.html)

### InnoDB

* [《一篇文章带你读懂Mysql和InnoDB》](https://my.oschina.net/kailuncen/blog/1504217)

### 优化

* [《MySQL36条军规》](http://vdisk.weibo.com/s/muWOT)

* [《MYSQL性能优化的最佳20+条经验》](https://www.cnblogs.com/zhouyusheng/p/8038224.html)

* [《SQL优化之道》](https://blog.csdn.net/when_less_is_more/article/details/70187459)

* [《mysql数据库死锁的产生原因及解决办法》](https://www.cnblogs.com/sivkun/p/7518540.html)

* [《导致索引失效的可能情况》](https://blog.csdn.net/monkey_d_feilong/article/details/52291556)

* [《 MYSQL分页limit速度太慢优化方法》](https://blog.csdn.net/zy_281870667/article/details/51604540)

  * 原则上就是缩小扫描范围。

### 索引

#### 聚集索引, 非聚集索引

* [《MySQL 聚集索引/非聚集索引简述》](https://blog.csdn.net/no_endless/article/details/77073549)
* [《MyISAM和InnoDB的索引实现》](https://www.cnblogs.com/zlcxbb/p/5757245.html)

MyISAM 是非聚集，InnoDB 是聚集

#### 复合索引

* [《复合索引的优点和注意事项》](https://www.cnblogs.com/summer0space/p/7247778.html)

  * 文中有一处错误：

  > 对于复合索引,在查询使用时,最好将条件顺序按找索引的顺序,这样效率最高; select \* from table1 where col1=A AND col2=B AND col3=D 如果使用 where col2=B AND col1=A 或者 where col2=B 将不会使用索引

  * 原文中提到索引是按照“col1，col2，col3”的顺序创建的，而mysql在按照最左前缀的索引匹配原则，且会自动优化 where 条件的顺序，当条件中只有 col2=B AND col1=A 时，会自动转化为 col1=A AND col2=B，所以依然会使用索引。

* [《MySQL查询where条件的顺序对查询效率的影响》](https://www.cnblogs.com/acode/p/7489258.html)

#### 自适应哈希索引\(AHI\)

* [《InnoDB存储引擎——自适应哈希索引》](https://blog.csdn.net/Linux_ever/article/details/62043708)

### explain

* [《MySQL 性能优化神器 Explain 使用分析》](https://segmentfault.com/a/1190000008131735)



