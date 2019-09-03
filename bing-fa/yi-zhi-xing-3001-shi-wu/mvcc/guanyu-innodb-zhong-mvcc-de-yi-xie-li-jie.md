# [关于innodb中MVCC的一些理解](https://www.cnblogs.com/chenpingzhao/p/5065316.html)

### **一、MVCC简介**

MVCC \(Multiversion Concurrency Control\)，即多版本并发控制技术,它使得大部分支持行锁的事务引擎，不再单纯的使用行锁来进行数据库的并发控制，取而代之的是把数据库的行锁与行的多个版本结合起来，只需要很小的开销,就可以实现非锁定读，从而大大提高数据库系统的并发性能

**读锁：**也叫共享锁、S锁，若事务T对数据对象A加上S锁，则事务T可以读A但不能修改A，其他事务只能再对A加S锁，而不能加X锁，直到T释放A上的S 锁。这保证了其他事务可以读A，但在T释放A上的S锁之前不能对A做任何修改。

**写锁：**又称排他锁、X锁。若事务T对数据对象A加上X锁，事务T可以读A也可以修改A，其他事务不能再对A加任何锁，直到T释放A上的锁。这保证了其他事务在T释放A上的锁之前不能再读取和修改A。

**表锁：**操作对象是数据表。Mysql大多数锁策略都支持\(常见mysql innodb\)，是系统开销最低但并发性最低的一个锁策略。事务t对整个表加读锁，则其他事务可读不可写，若加写锁，则其他事务增删改都不行。

**行级锁：**操作对象是数据表中的一行。是MVCC技术用的比较多的，但在MYISAM用不了，行级锁用mysql的储存引擎实现而不是mysql服务器。但行级锁对系统开销较大，处理高并发较好。

### 二、MVCC实现原理

innodb MVCC主要是为Repeatable-Read事务隔离级别做的。在此隔离级别下，A、B客户端所示的数据相互隔离，互相更新不可见

了解innodb的行结构、Read-View的结构对于理解innodb mvcc的实现由重要意义

innodb存储的最基本row中包含一些额外的存储信息 DATA\_TRX\_ID，DATA\_ROLL\_PTR，DB\_ROW\_ID，DELETE BIT

* 6字节的DATA\_TRX\_ID 标记了最新更新这条行记录的transaction id，每处理一个事务，其值自动+1

* 7字节的DATA\_ROLL\_PTR 指向当前记录项的rollback segment的undo log记录，找之前版本的数据就是通过这个指针

* 6字节的DB\_ROW\_ID，当由innodb自动产生聚集索引时，聚集索引包括这个DB\_ROW\_ID的值，否则聚集索引中不包括这个值.，这个用于索引当中
* DELETE BIT位用于标识该记录是否被删除，这里的不是真正的删除数据，而是标志出来的删除。真正意义的删除是在commit的时候



