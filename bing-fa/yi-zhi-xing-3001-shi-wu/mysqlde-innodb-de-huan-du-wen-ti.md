## [ ](http://dev.mysql.com/doc/refman/5.0/en/set-transaction.html)MySQL的InnoDB的幻读问题

[MySQL InnoDB事务的隔离级别](http://dev.mysql.com/doc/refman/5.0/en/set-transaction.html)有四级，默认是“可重复读”（REPEATABLE READ）。

* 未提交读（READ UNCOMMITTED）。另一个事务修改了数据，但尚未提交，而本事务中的SELECT会读到这些未被提交的数据（脏读）。
* 提交读（READ COMMITTED）。本事务读取到的是最新的数据（其他事务提交后的）。问题是，在同一个事务里，前后两次相同的SELECT会读到不同的结果（不重复读）。
* 可重复读（REPEATABLE READ）。在同一个事务里，SELECT的结果是事务开始时时间点的状态，因此，同样的SELECT操作读到的结果会是一致的。但是，会有幻读现象（稍后解释）。
* 串行化（SERIALIZABLE）。读操作会隐式获取共享锁，可以保证不同事务间的互斥。

四个级别逐渐增强，每个级别解决一个问题。

* 脏读，最容易理解。另一个事务修改了数据，但尚未提交，而本事务中的SELECT会读到这些未被提交的数据。
* 不重复读。解决了脏读后，会遇到，同一个事务执行过程中，另外一个事务提交了新数据，因此本事务先后两次读到的数据结果会不一致。
* 幻读。解决了不重复读，保证了同一个事务里，查询的结果都是事务开始时的状态（一致性）。但是，如果另一个事务同时提交了新数据，本事务再更新时，就会“惊奇的”发现了这些新数据，貌似之前读到的数据是“鬼影”一样的幻觉。

借鉴并改造了一个搞笑的比喻：

* 脏读。假如，中午去食堂打饭吃，看到一个座位被同学小Q占上了，就认为这个座位被占去了，就转身去找其他的座位。不料，这个同学小Q起身走了。事实：该同学小Q只是临时坐了一小下，并未“提交”。
* 不重复读。假如，中午去食堂打饭吃，看到一个座位是空的，便屁颠屁颠的去打饭，回来后却发现这个座位却被同学小Q占去了。
* 幻读。假如，中午去食堂打饭吃，看到一个座位是空的，便屁颠屁颠的去打饭，回来后，发现这些座位都还是空的（重复读），窃喜。走到跟前刚准备坐下时，却惊现一个恐龙妹，严重影响食欲。仿佛之前看到的空座位是“幻影”一样。

一些文章写到InnoDB的可重复读避免了“幻读”（phantom read），这个说法并不准确。

做个试验：\(以下所有试验要注意存储引擎和隔离级别\)

    mysql> show create table t_bitfly\G;
    CREATE TABLE `t_bitfly` (
    `id` bigint(20) NOT NULL default '0',
    `value` varchar(32) default NULL,
    PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=gbk

    mysql> select @@global.tx_isolation, @@tx_isolation;
    +-----------------------+-----------------+
    | @@global.tx_isolation | @@tx_isolation  |
    +-----------------------+-----------------+
    | REPEATABLE-READ       | REPEATABLE-READ |
    +-----------------------+-----------------+

试验一：

```
t Session A                   Session B
|
| START TRANSACTION;          START TRANSACTION;
|
| SELECT * FROM t_bitfly;
| empty set
|                             INSERT INTO t_bitfly
|                             VALUES (1, 'a');
|
| SELECT * FROM t_bitfly;
| empty set
|                             COMMIT;
|
| SELECT * FROM t_bitfly;
| empty set
|
| INSERT INTO t_bitfly VALUES (1, 'a');
| ERROR 1062 (23000):
| Duplicate entry '1' for key 1
v (shit, 刚刚明明告诉我没有这条记录的)
```

如此就出现了幻读，以为表里没有数据，其实数据已经存在了，傻乎乎的提交后，才发现数据冲突了。

试验二：

```
t Session A                  Session B
|
| START TRANSACTION;         START TRANSACTION;
|
| SELECT * FROM t_bitfly;
| +------+-------+
| | id   | value |
| +------+-------+
| |    1 | a     |
| +------+-------+
|                            INSERT INTO t_bitfly
|                            VALUES (2, 'b');
|
| SELECT * FROM t_bitfly;
| +------+-------+
| | id   | value |
| +------+-------+
| |    1 | a     |
| +------+-------+
|                            COMMIT;
|
| SELECT * FROM t_bitfly;
| +------+-------+
| | id   | value |
| +------+-------+
| |    1 | a     |
| +------+-------+
|
| UPDATE t_bitfly SET value='z';
| Rows matched: 2  Changed: 2  Warnings: 0
| (怎么多出来一行)
|
| SELECT * FROM t_bitfly;
| +------+-------+
| | id   | value |
| +------+-------+
| |    1 | z     |
| |    2 | z     |
| +------+-------+
|
v
```

本事务中第一次读取出一行，做了一次更新后，另一个事务里提交的数据就出现了。也可以看做是一种幻读。

------

那么，InnoDB指出的可以避免幻读是怎么回事呢？

