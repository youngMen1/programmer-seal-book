# 1.mysqlbinlog 工具分析binlog日志

## 1.1.初步了解binlog
1.MySQL的二进制日志binlog可以说是MySQL最重要的日志，它记录了所有的DDL和DML语句（除了数据查询语句select）,以事件形式记录，还包含语句所执行的消耗的时间，MySQL的二进制日志是事务安全型的。
* DDL

Data Definition Language 数据库定义语言

主要的命令有create、alter、drop等，ddl主要是用在定义或改变表(table)的结构,数据类型，表之间的连接和约束等初始工作上，他们大多在建表时候使用。

* DML

Data Manipulation Language 数据操纵语言

主要命令是slect,update,insert,delete,就像它的名字一样，这4条命令是用来对数据库里的数据进行操作的语言

2.mysqlbinlog常见的选项有一下几个：

a、--start-datetime：从二进制日志中读取指定等于时间戳或者晚于本地计算机的时间

b、--stop-datetime：从二进制日志中读取指定小于时间戳或者等于本地计算机的时间 取值和上述一样

c、--start-position：从二进制日志中读取指定position 事件位置作为开始。

d、--stop-position：从二进制日志中读取指定position 事件位置作为事件截至

3.一般来说开启binlog日志大概会有1%的性能损耗。

4.binlog日志有两个最重要的使用场景。

a、mysql主从复制：mysql replication在master端开启binlog,master把它的二进制日志传递给slaves来达到master-slave数据一致的目的。

b、数据恢复：通过mysqlbinlog工具来恢复数据。

binlog日志包括两类文件：

1)、二进制日志索引文件(文件名后缀为.index)用于记录所有的二进制文件。

2)、二进制日志文件(文件名后缀为.00000*)记录数据库所有的DDL和DML(除了数据查询语句select)语句事件。

## 1.2.开启binlog日志

1、编辑打开mysql配置文件/application/mysql3307/my.cnf在

[mysqld]区块添加

log-bin=mysql-bin(也可指定二进制日志生成的路径，如：log-bin=/opt/Data/mysql-bin)
server-id=1

binlog_format=MIXED(加入此参数才能记录到insert语句)


## 1.3.常用的binlog日志操作命令
## 1.4.查看binlog日志内容，常用有两种方式
## 1.5.利用binlog日志恢复mysql数据

# 2.总结

所谓恢复，就是让mysql将保存在binlog日志中指定段落区间的sql语句逐个重新执行一次而已。

# 3.参考
mysqlbinlog 工具分析binlog日志
https://www.cnblogs.com/lvzf/p/10689462.html
mysql binlog详解【重要而且更加详细】
https://www.cnblogs.com/Presley-lpc/p/9619571.html


