## 事务隔离
* 未提交读：一个事务可以读取另一个未提交的数据，容易出现脏读的情况。

* 读提交：一个事务等另外一个事务提交之后才可以读取数据，但会出现不可重复读的情况（多次读取的数据不一致），读取过程中出现UPDATE操作，会多。（大多数数据库默认级别是RC，比如SQL Server，Oracle），读取的时候不可以修改。

* 可重复读： 同一个事务里确保每次读取的时候，获得的是同样的数据，但不保障原始数据被其他事务更新（幻读），Mysql InnoDB 就是这个级别。

* 序列化：所有事物串行处理（牺牲了效率）

* [《理解事务的4种隔离级别》](https://blog.csdn.net/qq_33290787/article/details/51924963)

* [数据库事务的四大特性及事务隔离级别](https://www.cnblogs.com/z-sm/p/7245981.html)

* [《MySQL的InnoDB的幻读问题 》](http://blog.sina.com.cn/s/blog_499740cb0100ugs7.html)

  * 幻读的例子非常清楚。
  * 通过 SELECT ... FOR UPDATE 解决。

* [《一篇文章带你读懂MySQL和InnoDB》](https://draveness.me/mysql-innodb)

  * 图解脏读、不可重复读、幻读问题。



