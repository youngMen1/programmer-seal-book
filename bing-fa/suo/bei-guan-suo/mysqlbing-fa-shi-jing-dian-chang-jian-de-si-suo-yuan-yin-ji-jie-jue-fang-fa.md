# [Mysql并发时经典常见的死锁原因及解决方法](https://www.cnblogs.com/zejin2008/p/5262751.html)

**1.   mysql都有什么锁**

MySQL有三种锁的级别：页级、表级、行级。

表级锁：开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高,并发度最低。

行级锁：开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低,并发度也最高。

页面锁：开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般

算法：

next KeyLocks锁，同时锁住记录\(数据\)，并且锁住记录前面的Gap

Gap锁，不锁记录，仅仅记录前面的Gap

Recordlock锁（锁数据，不锁Gap）

所以其实 Next-KeyLocks=Gap锁+ Recordlock锁

**2.   什么情况下会造成死锁**

所谓死锁&lt;DeadLock&gt;: 是指两个或两个以上的进程在执行过程中,  
因争夺资源而造成的一种互相等待的现象,若无外力作用,它们都将无法推进下去.  
此时称系统处于死锁状态或系统产生了死锁,这些永远在互相等竺的进程称为死锁进程.  
表级锁不会产生死锁.所以解决死锁主要还是针对于最常用的InnoDB.

死锁的关键在于**：两个\(或以上\)的Session加锁的顺序不一致。**

那么对应的解决死锁问题的关键就是：让不同的session加锁有次序

**3.   一些常见的死锁案例**

**案例一：**

需求：将投资的钱拆成几份随机分配给借款人。

起初业务程序思路是这样的：

投资人投资后，将金额随机分为几份，然后随机从借款人表里面选几个，然后通过一条条select for update 去更新借款人表里面的余额等。

象出来就是一个session通过for循环会有几条如下的语句：

Select \* from xxx where id='随机id' for update

基本来说，程序开启后不一会就死锁。

这可以是说最经典的死锁情形了。

例如两个用户同时投资，A用户金额随机分为2份，分给借款人1，2

B用户金额随机分为2份，分给借款人2，1

由于加锁的顺序不一样，死锁当然很快就出现了。

**对于这个问题的改进很简单，直接把所有分配到的借款人直接一次锁住就行了。**

**Select \* from xxx where id in \(xx,xx,xx\) for update**

**在in里面的列表值mysql是会自动从小到大排序，加锁也是一条条从小到大加的锁**

```
以id为主键为例，目前还没有id=22的行

Session1:

select * from t3 where id=22 for update;

Empty set (0.00 sec)



session2:

select * from t3 where id=23  for update;

Empty set (0.00 sec)



Session1:

insert into t3 values(22,'ac','a',now());

锁等待中……



Session2:

insert into t3 values(23,'bc','b',now());

ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction
```

**案例2：**

在开发中，经常会做这类的判断需求：根据字段值查询（有索引），如果不存在，则插入；否则更新。

```
以id为主键为例，目前还没有id=22的行

Session1:

select * from t3 where id=22 for update;

Empty set (0.00 sec)



session2:

select * from t3 where id=23  for update;

Empty set (0.00 sec)



Session1:

insert into t3 values(22,'ac','a',now());

锁等待中……



Session2:

insert into t3 values(23,'bc','b',now());

ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction
```

当对存在的行进行锁的时候\(主键\)，mysql就只有行锁。

当对未存在的行进行锁的时候\(即使条件为主键\)，mysql是会锁住一段范围（有gap锁）





锁住的范围为：

\(无穷小或小于表中锁住id的最大值，无穷大或大于表中锁住id的最小值\)



如：如果表中目前有已有的id为（11 ， 12）

那么就锁住（12，无穷大）

如果表中目前已有的id为（11 ， 30）

那么就锁住（11，30）



**对于这种死锁的解决办法是：**

**insert into t3\(xx,xx\) on duplicate key update \`xx\`='XX';**



用mysql特有的语法来解决此问题。因为insert语句对于主键来说，插入的行不管有没有存在，都会只有行锁。

