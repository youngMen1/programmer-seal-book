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

查看事务具体状态的话，你可以查 information_schema 库的 innodb_trx 表。

```
select * from information_schema.innodb_trx
```

## 1.3.全局读锁
MySQL 提供了一个加全局读锁的方法，命令是：
`Flush tables with read lock` 
官方自带的逻辑备份工具是 mysqldump。当 mysqldump 使用参数–single-transaction 的时候，导数据之前就会启动一个事务，来确保拿到一致性视图。而由于 MVCC 的支持，这个过程中数据是可以正常更新的。



