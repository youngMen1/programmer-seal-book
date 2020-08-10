# 1.mysqlbinlog 工具分析binlog日志

## 1.1.初步了解binlog
1.MySQL的二进制日志binlog可以说是MySQL最重要的日志，它记录了所有的DDL和DML语句（除了数据查询语句select）,以事件形式记录，还包含语句所执行的消耗的时间，MySQL的二进制日志是事务安全型的。
* DDL

Data Definition Language 数据库定义语言

主要的命令有create、alter、drop等，ddl主要是用在定义或改变表(table)的结构,数据类型，表之间的连接和约束等初始工作上，他们大多在建表时候使用。

* DML

Data Manipulation Language 数据操纵语言

主要命令是slect,update,insert,delete,就像它的名字一样，这4条命令是用来对数据库里的数据进行操作的语言


## 1.2.开启binlog日志
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


