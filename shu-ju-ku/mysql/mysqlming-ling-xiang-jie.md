# 1.MySQL命令详解

## 1.1.show processlist

```
show processlist;
```

![](/static/image/微信截图_20200717173429.png)

SHOW PROCESSLIST显示哪些线程正在运行

如果您有root权限，您可以看到所有线程。否则，您只能看到登录的用户自己的线程，通常只会显示100条如果想看跟多的可以使用full修饰（show full processlist）

**说明各列的含义和用途:**  
id：ID标识，要kill一个语句的时候很有用  
use：  当前连接用户  
host：显示这个连接从哪个ip的哪个端口上发出  
db：数据库名  
command： 连接状态，一般是休眠（sleep），查询（query），连接（connect）  
time：连接持续时间，单位是秒  
state：显示当前sql语句的状态  
info：显示这个sql语句

其中state的状态十分关键，下表列出state主要状态和描述：  
![](/static/image/微信截图_20200717173950.png)

## 1.2.show variables like 'transaction\_isolation';

可以用 show variables 来查看当前事务的隔离级别

```
show variables like 'transaction_isolation';
```

## 1.2.查询长事务

你可以在 information\_schema 库的 innodb\_trx 这个表中查询长事务，比如下面这个语句，用于查找持续时间超过 60s 的事务。

```
select * from information_schema.innodb_trx where TIME_TO_SEC(timediff(now(),trx_started))>60
```

查看事务具体状态的话，你可以查 information\_schema 库的 innodb\_trx 表。

```
select * from information_schema.innodb_trx
```

## 1.3.全局读锁

MySQL 提供了一个加全局读锁的方法，命令是：  
`Flush tables with read lock`  
官方自带的逻辑备份工具是 mysqldump。当 mysqldump 使用参数–single-transaction 的时候，导数据之前就会启动一个事务，来确保拿到一致性视图。而由于 MVCC 的支持，这个过程中数据是可以正常更新的。

为什么不使用 set global readonly=true 的方式呢？确实 readonly 方式也可以让全库进入只读状态，但我还是会建议你用 FTWRL 方式，主要有两个原因：

* 一是，在有些系统中，readonly 的值会被用来做其他逻辑，比如用来判断一个库是主库还是备库。因此，修改 global 变量的方式影响面更大，我不建议你使用。
* 二是，在异常处理机制上有差异。如果执行 FTWRL 命令之后由于客户端发生异常断开，那么 MySQL 会自动释放这个全局锁，整个库回到可以正常更新的状态。而将整个库设置为 readonly 之后，如果客户端发生异常，则数据库就会一直保持 readonly 状态，这样会导致整个库长时间处于不可写状态，风险较高。

## 1.4.show warnings;

SHOW WARNINGS是诊断语句，显示有关在当前会话中执行语句所导致的条件（错误，警告和注释）的信息。警告DML语句诸如产生INSERT，UPDATE和LOAD DATA以及DDL语句如CREATE TABLE和ALTER TABLE。

## 1.5.show grants 命令查看账户的权限



