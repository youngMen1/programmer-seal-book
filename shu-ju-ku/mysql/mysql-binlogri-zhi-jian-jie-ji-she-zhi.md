# 1.MySQL - binlog日志简介及设置

mysql-binlog是MySQL数据库的二进制日志，用于记录用户对数据库操作的SQL语句（\(除了数据查询语句）信息。可以使用mysqlbin命令查看二进制日志的内容。



## 1.1.MySQL binlog格式

binlog的格式也有三种：STATEMENT、ROW、MIXED 。


1、STATMENT模式：基于SQL语句的复制(statement-based replication, SBR)，每一条会修改数据的sql语句会记录到binlog中。


优点：不需要记录每一条SQL语句与每行的数据变化，这样子binlog的日志也会比较少，减少了磁盘IO，提高性能。


缺点：在某些情况下会导致master-slave中的数据不一致(如sleep()函数， last_insert_id()，以及user-defined functions(udf)等会出现问题)


2、基于行的复制(row-based replication, RBR)：不记录每一条SQL语句的上下文信息，仅需记录哪条数据被修改了，修改成了什么样子了。


优点：不会出现某些特定情况下的存储过程、或function、或trigger的调用和触发无法被正确复制的问题。


缺点：会产生大量的日志，尤其是alter table的时候会让日志暴涨。


3、混合模式复制(mixed-based replication, MBR)：以上两种模式的混合使用，一般的复制使用STATEMENT模式保存binlog，对于STATEMENT模式无法复制的操作使用ROW模式保存binlog，MySQL会根据执行的SQL语句选择日志保存方式。

## 1.2.binlog配置

在MySQL配置文件my.cnf文件中的mysqld节中添加下面的配置文件：


[mysqld]


### 设置日志格式

binlog_format = mixed


### 设置日志路径，注意路经需要mysql用户有权限写

log-bin = /data/mysql/logs/mysql-bin.log


### 设置binlog清理时间

expire_logs_days = 7


### binlog每个日志文件大小

max_binlog_size = 100m


### binlog缓存大小

binlog_cache_size = 4m


### 最大binlog缓存大小

max_binlog_cache_size = 512m



重启MySQL生效，如果不方便重启服务，也可以直接修改对应的变量即可。


# 2.总结

无论是增量备份还是主从复制，都是需要开启mysql-binlog日志，最好跟数据目录设置到不同的磁盘分区，可以降低io等待，提升性能；并且在磁盘故障的时候可以利用mysql-binlog恢复数据。
![](/static/image/u=1778793404,3158621773&fm=173&app=25&f=JPEG.jpg)


