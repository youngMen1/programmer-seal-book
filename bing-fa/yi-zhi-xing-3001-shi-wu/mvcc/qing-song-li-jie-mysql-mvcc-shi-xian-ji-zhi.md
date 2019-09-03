# 轻松理解MYSQL MVCC 实现机制

1. MVCC简介

1.1 什么是MVCC

MVCC是一种多版本并发控制机制。

1.2 MVCC是为了解决什么问题?

大多数的MYSQL事务型存储引擎,如,InnoDB，Falcon以及PBXT都不使用一种简单的行锁机制.事实上,他们都和MVCC–多版本并发控制来一起使用.

大家都应该知道,锁机制可以控制并发操作,但是其系统开销较大,而MVCC可以在大多数情况下代替行级锁,使用MVCC,能降低其系统开销.

1.3 MVCC实现

MVCC是通过保存数据在某个时间点的快照来实现的. 不同存储引擎的MVCC. 不同存储引擎的MVCC实现是不同的,典型的有乐观并发控制和悲观并发控制.

2.MVCC 具体实现分析

下面,我们通过InnoDB的MVCC实现来分析MVCC使怎样进行并发控制的.

InnoDB的MVCC,是通过在每行记录后面保存两个隐藏的列来实现的,这两个列，分别保存了这个行的创建时间，一个保存的是行的删除时间。这里存储的并不是实际的时间值,而是系统版本号\(可以理解为事务的ID\)，没开始一个新的事务，系统版本号就会自动递增，事务开始时刻的系统版本号会作为事务的ID.下面看一下在REPEATABLE READ隔离级别下,MVCC具体是如何操作的.

2.1简单的小例子

create table yang\(

id int primary key auto\_increment,

name varchar\(20\)\);

```
假设系统的版本号从1开始.
```

假设系统的版本号从1开始.

INSERT

InnoDB为新插入的每一行保存当前系统版本号作为版本号.

第一个事务ID为1；

```
start transaction;
insert into yang values(NULL,'yang') ;
insert into yang values(NULL,'long');
insert into yang values(NULL,'fei');
commit;
```

对应在数据中的表如下\(后面两列是隐藏列,我们通过查询语句并看不到\)

| id | name | 创建时间\(事务ID\) | 删除时间\(事务ID\) |
| :--- | :--- | :--- | :--- |
| 1 | yang | 1 | undefined |
| 2 | long | 1 | undefined |
| 3 | fei | 1 | undefined |

SELECT

InnoDB会根据以下两个条件检查每行记录:

a.InnoDB只会查找版本早于当前事务版本的数据行\(也就是,行的系统版本号小于或等于事务的系统版本号\)，这样可以确保事务读取的行，要么是在事务开始前已经存在的，要么是事务自身插入或者修改过的.

b.行的删除版本要么未定义,要么大于当前事务版本号,这可以确保事务读取到的行，在事务开始之前未被删除.

只有a,b同时满足的记录，才能返回作为查询结果.

DELETE

InnoDB会为删除的每一行保存当前系统的版本号\(事务的ID\)作为删除标识.

看下面的具体例子分析:

第二个事务,ID为2;

```
start transaction;
select * from yang;  //(1)
select * from yang;  //(2)
commit;
```

#### 假设1 {#假设1}

假设在执行这个事务ID为2的过程中,刚执行到\(1\),这时,有另一个事务ID为3往这个表里插入了一条数据;  
第三个事务ID为3;

