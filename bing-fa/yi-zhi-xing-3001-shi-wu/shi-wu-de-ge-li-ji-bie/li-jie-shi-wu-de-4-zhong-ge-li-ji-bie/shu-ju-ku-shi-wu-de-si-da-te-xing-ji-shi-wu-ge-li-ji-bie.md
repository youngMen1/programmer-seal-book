# 1.[数据库事务的四大特性及事务隔离级别](https://www.cnblogs.com/z-sm/p/7245981.html)

## 1.1.概要

**事务的四个特性：**原子性、一致性、隔离性、持久性

**事务不隔离带来的问题：**更新丢失、脏读、不可重复读、虚读（幻读）。其中更新丢失就是并发写，这是一定不允许的，因此一定要解决更新丢失问题。

**事务隔离的级别：**读未提交（1000）、读已提交（1100）、可重复读（1110）、串行化（1111）。

|  | 更新丢失 | 脏读 | 不可重复读 | 幻读 |
| :--- | :--- | :--- | :--- | :--- |
| RU（读未提交） | 避免 |  |  |  |
| RC（读提交） | 避免 | 避免 |  |  |
| RR（可重复读） | 避免 | 避免 | 避免 |  |
| S（串行化） | 避免 | 避免 | 避免 | 避免 |

## 1.2.事务四个特性 {#blogTitle0}

如果一个数据库声称支持事务的操作，那么该数据库必须要具备以下四个特性：

### 原子性（Atomicity） {#blogTitle1}

原子性是指事务包含的所有操作要么全部成功，要么全部失败回滚，这和前面两篇博客介绍事务的功能是一样的概念，因此事务的操作如果成功就必须要完全应用到数据库，如果操作失败则不能对数据库有任何影响。

### 一致性（Consistency） {#blogTitle2}

一致性是指事务必须使数据库从一个一致性状态变换到另一个一致性状态，也就是说一个事务执行之前和执行之后都必须处于一致性状态。

拿转账来说，假设用户A和用户B两者的钱加起来一共是5000，那么不管A和B之间如何转账，转几次账，事务结束后两个用户的钱相加起来应该还得是5000，这就是事务的一致性。

### 隔离性（Isolation） {#blogTitle3}

隔离性是当多个用户并发访问数据库时，比如操作同一张表时，数据库为每一个用户开启的事务，不能被其他事务的操作所干扰，多个并发事务之间要相互隔离。

即要达到这么一种效果：对于任意两个并发的事务T1和T2，在事务T1看来，T2要么在T1开始之前就已经结束，要么在T1结束之后才开始，这样每个事务都感觉不到有其他事务在并发地执行。

关于事务的隔离性数据库提供了多种隔离级别，稍后会介绍到。

### 持久性（Durability） {#blogTitle4}

持久性是指一个事务一旦被提交了，那么对数据库中的数据的改变就是永久性的，即便是在数据库系统遇到故障的情况下也不会丢失提交事务的操作。

例如我们在使用JDBC操作数据库时，在提交事务方法后，提示用户事务操作完成，当我们程序执行完成直到看到提示后，就可以认定事务以及正确提交，即使这时候数据库出现了问题，也必须要将我们的事务完全执行完成，否则就会造成我们看到提示事务处理完毕，但是数据库因为故障而没有执行事务的重大错误。

## 1.3.事务隔离 {#blogTitle5}

以上介绍完事务的四大特性\(简称ACID\)，现在重点来说明下事务的隔离性，当多个线程都开启事务操作数据库中的数据时，数据库系统要能进行隔离操作，以保证各个线程获取数据的准确性。

### 1.3.1.不事务隔离带来的问题 {#blogTitle6}

在介绍数据库提供的各种隔离级别之前，我们先看看如果不考虑事务的隔离性，会发生的几种问题：

**更新丢失**：两事务同时更新，一个失败回滚覆盖另一个事务的更新。或事务1执行更细操作，在事务1结束前事务2也更新，则事务1的更细结果被事务2的覆盖了。

**脏读**：事务T2读取到事务T1修改了但是还未提交的数据，之后事务T1又回滚其更新操作，导致事务T2读到的是脏数据。

**不可重复读**：事务T1读取某个数据后，事务T2对其做了修改，当事务T1再次读该数据时得到与前一次不同的值。

**虚读（幻读）**：事务T1读取在读取某范围数据时，事务T2又插入一条数据，当事务T1再次数据这个范围数据时发现不一样了，出现了一些“幻影行”。

脏读和不可重复读的区别：脏读是某一事务读取了另一个事务未提交的脏数据，而不可重复读则是读取了前一事务提交的数据。

不可重复读和幻读的异同：都是读取了另一条已经提交的事务（这点就脏读不同），所不同的是不可重复读查询的都是同一个数据项，而幻读针对的是一批数据整体（比如数据的个数）。

### 1.3.2.更新丢失 {#blogTitle7}

**1、更新丢失（Lostupdate）**

两个事务同时更新，第二个事务回滚会覆盖第一个事务更新的数据，导致更新丢失

**2、两次更新问题（Secondlost updates problem）**

两个事务都读取了数据，并同时更新，第一个事务更新失败，因为被第二个事务覆盖。

### 1.3.3.脏读 {#blogTitle8}

脏读是指在一个事务处理过程里读取了另一个未提交的事务中的数据。

当一个事务正在多次修改某个数据，而在这个事务中这多次的修改都还未提交，这时一个并发的事务来访问该数据，就会造成两个事务得到的数据不一致。例如：用户A向用户B转账100元，对应SQL命令如下：

```
update account set money=money+100 where name=’B’;  (此时A通知B)

update account set money=money - 100 where name=’A’;
```

当只执行第一条SQL时，A通知B查看账户，B发现确实钱已到账（此时即发生了脏读），而之后无论第二条SQL是否执行，只要该事务不提交，则所有操作都将回滚，那么当B以后再次查看账户时就会发现钱其实并没有转。

### 1.3.4.不可重复读 {#blogTitle9}

不可重复读是指在对于数据库中的某个数据，一个事务范围内多次查询却返回了不同的数据值，这是由于在查询间隔，被另一个事务修改并提交了。

例如事务T1在读取某一数据，而事务T2立马修改了这个数据并且提交事务给数据库，事务T1再次读取该数据就得到了不同的结果，发送了不可重复读。

不可重复读和脏读的区别是，脏读是某一事务读取了另一个事务未提交的脏数据，而不可重复读则是读取了前一事务提交的数据。

**在某些情况下，不可重复读并不是问题**，比如我们多次查询某个数据当然以最后查询得到的结果为主。但在另一些情况下就有可能发生问题，例如对于同一个数据A和B依次查询就可能不同，A和B就可能打起来了……

### 1.3.5.虚读\(幻读\) {#blogTitle10}

幻读是事务非独立执行时发生的一种现象。例如事务T1对一个表中所有的行的某个数据项做了从“1”修改为“2”的操作，这时事务T2又对这个表中插入了一行数据项，而这个数据项的数值还是为“1”并且提交给数据库。而操作事务T1的用户如果再查看刚刚修改的数据，会发现还有一行没有修改，其实这行是从事务T2中添加的，就好像产生幻觉一样，这就是发生了幻读。

幻读和不可重复读的异同：都是读取了另一条已经提交的事务（这点就脏读不同），所不同的是不可重复读查询的都是同一个数据项，而幻读针对的是一批数据整体（比如数据的个数）。

## 1.4.事务隔离的级别 {#blogTitle11}

为此我们需要通过提供不同类型的“锁”机制针对数据库事务进行不同程度的并发访问控制，由此产生了不同的事务隔离级别：隔离级别（低-&gt;高）。SQL、SQL2标准定义了四种隔离级别：

**● 读未提交（Read Uncommitted）**

含义解释：只限制同一数据写事务禁止其他写事务。解决”更新丢失”。（**一事务写时禁止其他事务写**）

名称解释：可读取未提交数据

所需的锁：排他写锁

**● 读提交（Read Committed）**

含义解释：只限制同一数据写事务禁止其它读写事务。解决”脏读”，以及”更新丢失”。（**一事务写时禁止其他事务读写**）

名称解释：必须提交以后的数据才能被读取

所需的锁：排他写锁、瞬间共享读锁

**● 可重复读（Repeatable Read）**

含义解释：限制同一数据写事务禁止其他读写事务，读事务禁止其它写事务\(允许读\)。解决”不可重复读”，以及”更新丢失”和”脏读”。（**一事务写时禁止其他事务读写、一事务读时禁止其他事务**写）

注意没有解决幻读，解决幻读的方法是增加范围锁（range lock）或者表锁。

名称解释：能够重复读取

所需的锁：排他写锁、共享读锁

**● 串行化（Serializable）**

含义解释：限制所有读写事务都必须串行化实行。它要求事务序列化执行，事务只能一个接着一个地执行，但不能并发执行。如果仅仅通过“行级锁”是无法实现事务序列化的，必须通过其他机制保证新插入的数据不会被刚执行查询操作的事务访问到。（**一事务写时禁止其他事务读写、一事务读时禁止其他事务读写**）

所须的锁：范围锁或表锁

**下表是各隔离级别对各种异常的控制能力。**

|  | 更新丢失 | 脏读 | 不可重复读 | 幻读 |
| :--- | :--- | :--- | :--- | :--- |
| RU（读未提交） | 避免 |  |  |  |
| RC（读提交） | 避免 | 避免 |  |  |
| RR（可重复读） | 避免 | 避免 | 避免 |  |
| S（串行化） | 避免 | 避免 | 避免 | 避免 |

以上四种隔离级别最高的是Serializable级别，最低的是Read uncommitted级别，当然级别越高，数据完整性越好，但执行效率就越低。像Serializable这样的级别，就是以锁表的方式\(类似于Java多线程中的锁\)使得其他的线程只能在锁外等待，所以平时选用何种隔离级别应该根据实际情况。

## 1.5.常见数据库的事务隔离 {#blogTitle12}

| 数据库 | 默认级别 |
| :--- | :--- |
| MySQL | 可重复读（Repeatable Read） |
| Oracle | 读提交（Read Committed） |
| SQLServer | 读提交（Read Committed） |
| DB2 | 读提交（Read Committed） |
| PostgreSQL | 读提交（Read Committed） |

在MySQL数据库中，支持上面四种隔离级别，默认的为Repeatable read \(可重复读\)；此外，**MySQL的Repeatable Read隔离级别也解决了幻读问题**（通过Next-key lock加锁方法即范围锁解决不可重复读和幻读问题，如select \* from t where a&gt;10会对key为\[10,infinite）范围的行加锁，这样其他事务就不能对此范围内key对应的行更改）达到了SQL、SQL2标准中的Serializable级别。

在Oracle数据库中，只支持Serializable \(串行化\)级别和Read committed \(读已提交\)这两种级别，其中默认的为Read committed级别。

在MySQL数据库中查看当前事务的隔离级别：

```
select @@tx_isolation;
```

在MySQL数据库中设置事务的隔离 级别：

```
set  [glogal | session]  transaction isolation level 隔离级别名称; 

//设置全部连接或当前连接的事务隔离级别
set tx_isolation=’隔离级别名称; //设置当前连接的事务隔离级别
```

例1：查看当前事务的隔离级别：

![img](/static/image/787876-20160313202200007-1111796802.png)

例2：将事务的隔离级别设置为Read uncommitted级别：

![img](/static/image/787876-20160313202224241-2101542210.png)

或：

![img](/static/image/787876-20160313202245210-345198166.png)

记住：设置数据库的隔离级别一定要是在开启事务之前！

如果是使用JDBC对数据库的事务设置隔离级别的话，也应该是在调用Connection对象的setAutoCommit\(false\)方法之前。调用Connection对象的setTransactionIsolation\(level\)即可设置当前链接的隔离级别，至于参数level，可以使用Connection对象的字段：

![img](/static/image/787876-20160313202333460-377269897.png)

在JDBC中设置隔离级别的部分代码：

![img](/static/image/787876-20160313202355694-2106931487.png)

# 2.总结

隔离级别的设置只对当前链接有效，对于使用MySQL命令窗口而言，一个窗口就相当于一个链接，当前窗口设置的隔离级别只对当前窗口中的事务有效；对于JDBC操作数据库来说，一个Connection对象相当于一个链接，而对于Connection对象设置的隔离级别只对该Connection对象有效，与其他链接Connection对象无关。

