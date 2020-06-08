# 1.MySQL的InnoDB的幻读问题

[MySQL InnoDB事务的隔离级别](http://dev.mysql.com/doc/refman/5.0/en/set-transaction.html)有四级，**默认是“可重复读”（REPEATABLE READ）**。

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

---

那么，InnoDB指出的可以避免幻读是怎么回事呢？

```
http://dev.mysql.com/doc/refman/5.0/en/innodb-record-level-locks.html

By default, InnoDB operates in REPEATABLE READ transaction isolation level and with the innodb_locks_unsafe_for_binlog system variable disabled. In this case, InnoDB uses next-key locks for searches and index scans, which prevents phantom rows (see Section 13.6.8.5, “Avoiding the Phantom Problem Using Next-Key Locking”).
```

准备的理解是，当隔离级别是可重复读，且禁用innodb\_locks\_unsafe\_for\_binlog的情况下，在搜索和扫描index的时候使用的next-key locks可以避免幻读。

关键点在于，是InnoDB默认对一个普通的查询也会加next-key locks，还是说需要应用自己来加锁呢？如果单看这一句，可能会以为InnoDB对普通的查询也加了锁，如果是，那和序列化（SERIALIZABLE）的区别又在哪里呢？

MySQL manual里还有一段：

```
13.2.8.5. Avoiding the Phantom Problem Using Next-Key Locking (http://dev.mysql.com/doc/refman/5.0/en/innodb-next-key-locking.html)

To prevent phantoms, InnoDB uses an algorithm called next-key locking that combines index-row locking with gap locking.

You can use next-key locking to implement a uniqueness check in your application: If you read your data in share mode and do not see a duplicate for a row you are going to insert, then you can safely insert your row and know that the next-key lock set on the successor of your row during the read prevents anyone meanwhile inserting a duplicate for your row. Thus, the next-key locking enables you to “lock” the nonexistence of something in your table.
```

我的理解是说，InnoDB提供了next-key locks，但需要应用程序自己去加锁。manual里提供一个例子：

```
SELECT * FROM child WHERE id > 100 FOR UPDATE;
```

这样，InnoDB会给id大于100的行（假如child表里有一行id为102），以及100-102，102+的gap都加上锁。

可以使用show innodb status来查看是否给表加上了锁。

再看一个实验，要注意，表t\_bitfly里的id为主键字段。实验三：

```
t Session A                 Session B
|
| START TRANSACTION;        START TRANSACTION;
|
| SELECT * FROM t_bitfly
| WHERE id&lt;=1
| FOR UPDATE;
| +------+-------+
| | id   | value |
| +------+-------+
| |    1 | a     |
| +------+-------+
|                           INSERT INTO t_bitfly
|                           VALUES (2, 'b');
|                           Query OK, 1 row affected
|
| SELECT * FROM t_bitfly;
| +------+-------+
| | id   | value |
| +------+-------+
| |    1 | a     |
| +------+-------+
|                           INSERT INTO t_bitfly
|                           VALUES (0, '0');
|                           (waiting for lock ...
|                           then timeout)
|                           ERROR 1205 (HY000):
|                           Lock wait timeout exceeded;
|                           try restarting transaction
|
| SELECT * FROM t_bitfly;
| +------+-------+
| | id   | value |
| +------+-------+
| |    1 | a     |
| +------+-------+
|                           COMMIT;
|
| SELECT * FROM t_bitfly;
| +------+-------+
| | id   | value |
| +------+-------+
| |    1 | a     |
| +------+-------+
v
```

可以看到，用id&lt;=1加的锁，只锁住了id&lt;=1的范围，可以成功添加id为2的记录，添加id为0的记录时就会等待锁的释放。

MySQL manual里对可重复读里的锁的详细解释：

```
http://dev.mysql.com/doc/refman/5.0/en/set-transaction.html#isolevel_repeatable-read

For locking reads (SELECT with FOR UPDATE or LOCK IN SHARE MODE),UPDATE, and DELETE statements, locking depends on whether the statement uses a unique index with a unique search condition, or a range-type search condition. For a unique index with a unique search condition, InnoDB locks only the index record found, not the gap before it. For other search conditions, InnoDB locks the index range scanned, using gap locks or next-key (gap plus index-record) locks to block insertions by other sessions into the gaps covered by the range.
```

---

一致性读和提交读，先看实验，实验四：

```
t Session A                      Session B
|
| START TRANSACTION;             START TRANSACTION;
|
| SELECT * FROM t_bitfly;
| +----+-------+
| | id | value |
| +----+-------+
| |  1 | a     |
| +----+-------+
|                                INSERT INTO t_bitfly
|                                VALUES (2, 'b');
|                                COMMIT;
|
| SELECT * FROM t_bitfly;
| +----+-------+
| | id | value |
| +----+-------+
| |  1 | a     |
| +----+-------+
|
| SELECT * FROM t_bitfly LOCK IN SHARE MODE;
| +----+-------+
| | id | value |
| +----+-------+
| |  1 | a     |
| |  2 | b     |
| +----+-------+
|
| SELECT * FROM t_bitfly FOR UPDATE;
| +----+-------+
| | id | value |
| +----+-------+
| |  1 | a     |
| |  2 | b     |
| +----+-------+
|
| SELECT * FROM t_bitfly;
| +----+-------+
| | id | value |
| +----+-------+
| |  1 | a     |
| +----+-------+
v
```

如果使用普通的读，会得到一致性的结果，如果使用了加锁的读，就会读到“最新的”“提交”读的结果。

本身，可重复读和提交读是矛盾的。在同一个事务里，如果保证了可重复读，就会看不到其他事务的提交，违背了提交读；如果保证了提交读，就会导致前后两次读到的结果不一致，违背了可重复读。

可以这么讲，InnoDB提供了这样的机制，在默认的可重复读的隔离级别里，可以使用加锁读去查询最新的数据。

```
http://dev.mysql.com/doc/refman/5.0/en/innodb-consistent-read.html

If you want to see the “freshest” state of the database, you should use either the READ COMMITTED isolation level or a locking read:
SELECT * FROM t_bitfly LOCK IN SHARE MODE;
```

---

# 2.结论

MySQL InnoDB的可重复读并不保证避免幻读，需要应用使用加锁读来保证。而这个加锁度使用到的机制就是next-key locks。

