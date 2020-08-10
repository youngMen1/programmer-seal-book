# 1.MySQL - binlog日志简介及设置

mysql-binlog是MySQL数据库的二进制日志，用于记录用户对数据库操作的SQL语句（\(除了数据查询语句）信息。可以使用mysqlbin命令查看二进制日志的内容。



## MySQL binlog格式

binlog的格式也有三种：STATEMENT、ROW、MIXED 。


1、STATMENT模式：基于SQL语句的复制(statement-based replication, SBR)，每一条会修改数据的sql语句会记录到binlog中。


优点：不需要记录每一条SQL语句与每行的数据变化，这样子binlog的日志也会比较少，减少了磁盘IO，提高性能。


缺点：在某些情况下会导致master-slave中的数据不一致(如sleep()函数， last_insert_id()，以及user-defined functions(udf)等会出现问题)


2、基于行的复制(row-based replication, RBR)：不记录每一条SQL语句的上下文信息，仅需记录哪条数据被修改了，修改成了什么样子了。


优点：不会出现某些特定情况下的存储过程、或function、或trigger的调用和触发无法被正确复制的问题。


缺点：会产生大量的日志，尤其是alter table的时候会让日志暴涨。


3、混合模式复制(mixed-based replication, MBR)：以上两种模式的混合使用，一般的复制使用STATEMENT模式保存binlog，对于STATEMENT模式无法复制的操作使用ROW模式保存binlog，MySQL会根据执行的SQL语句选择日志保存方式。