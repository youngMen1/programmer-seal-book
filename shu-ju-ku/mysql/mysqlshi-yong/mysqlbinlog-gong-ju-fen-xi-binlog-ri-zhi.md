# 1.mysqlbinlog 工具分析binlog日志

## 1.1.初步了解binlog

1.MySQL的二进制日志binlog可以说是MySQL最重要的日志，它记录了所有的DDL和DML语句（除了数据查询语句select）,以事件形式记录，还包含语句所执行的消耗的时间，MySQL的二进制日志是事务安全型的。

* DDL

Data Definition Language 数据库定义语言

主要的命令有create、alter、drop等，ddl主要是用在定义或改变表\(table\)的结构,数据类型，表之间的连接和约束等初始工作上，他们大多在建表时候使用。

* DML

Data Manipulation Language 数据操纵语言

主要命令是slect,update,insert,delete,就像它的名字一样，这4条命令是用来对数据库里的数据进行操作的语言

2.mysqlbinlog常见的选项有一下几个：

* a、--start-datetime：从二进制日志中读取指定等于时间戳或者晚于本地计算机的时间

* b、--stop-datetime：从二进制日志中读取指定小于时间戳或者等于本地计算机的时间 取值和上述一样

* c、--start-position：从二进制日志中读取指定position 事件位置作为开始。

* d、--stop-position：从二进制日志中读取指定position 事件位置作为事件截至

3.一般来说开启binlog日志大概会有1%的性能损耗。

4.binlog日志有两个最重要的使用场景。

* a、mysql主从复制：mysql replication在master端开启binlog,master把它的二进制日志传递给slaves来达到master-slave数据一致的目的。

* b、数据恢复：通过mysqlbinlog工具来恢复数据。

binlog日志包括两类文件：

1\)、二进制日志索引文件\(文件名后缀为.index\)用于记录所有的二进制文件。

2\)、二进制日志文件\(文件名后缀为.00000\*\)记录数据库所有的DDL和DML\(除了数据查询语句select\)语句事件。

## 1.2.开启binlog日志

**binlog的格式也有三种：STATEMENT、ROW、MIXED 。**

1、编辑打开mysql配置文件/application/mysql3307/my.cnf在

\[mysqld\]区块添加

log-bin=mysql-bin\(也可指定二进制日志生成的路径，如：log-bin=/opt/Data/mysql-bin\)  
server-id=1

binlog\_format=MIXED\(加入此参数才能记录到insert语句\)

2、重启mysqld服务

```
/application/mysql3307/bin/mysqladmin -uroot -S /application/mysql3307/logs/mysql.sock -p shutdown

nohup /application/mysql3307/bin/mysqld_safe --defaults-file=/application/mysql3307/my.cnf --user=mysql &
```

3、查看binlog日志是否开启

`show variables like 'log_%'`

mysql&gt; show variables like 'log\_%';  
![](/static/image/1414258-20180910150727415-1015390112.png)

## 1.3.常用的binlog日志操作命令

1、查看所有binlog日志列表

`show master logs`

![](/static/image/1414258-20180910150911389-1398860498.png)

2、查看master状态，即最后（最新）一个binlog日志的编号名称，及其最后一个操作事件pos结束点\(Position\)值。

`show master status`

![](/static/image/1414258-20180910171026280-1049764479.png)

3、flush 刷新log日志，自此刻开始产生一个新编号的binlog日志文件;

`flush logs`

**注意：**每当mysqld服务重启时，会自动执行此命令，刷新binlog日志；在mysqlddump备份数据时加-F选项也会刷新binlog日志；

4、重置（清空）所有binlog日志

`reset master`

## 1.4.查看binlog日志内容，常用有两种方式

1、使用mysqlbinlog自带查看命令法

**注意：**

a、binlog是二进制文件，普通文件查看器cat、more、vim等都无法打开，必须使用自带的mysqlbinlog命令查看。

b、binlog日志与数据库文件在同目录中。

c、在Mysql5.5以下版本使用mysqlbinlog命令时如果报错，就加上"--no-defaults"选项

d、使用mysqlbinlog命令查看binlog日志内容，下面截取其中的一个片段分析分析：

![](/static/image/1414258-20180910173629464-1243321061.png)

**解释：**

server id 1:数据库主机的服务号

end\_log\_pos 796 :sql结束时的pos节点

thread\_id=11:线程号

e、也可根据时间点查看

```
/home/software/mysql-5.1.72-linux-x86_64-glibc23/bin/mysqlbinlog --no-defaults mysql-bin.000720 --start-datetime="2018-09-12 18:45:00" --stop-datetime="2018-09-12:18:47:00"
```

2、上面这种办法读取出binlog日志的全文内容比较多，不容易分辨查看到pos点信息，下面介绍一种更为方便的查询命令：

```
mysql> show binlog events [IN 'log_name'] [FROM pos] [LIMIT [offset,] row_count];
```

**参数解释：**

a、IN 'log\_name':指定要查询的binlog文件名（不指定就是第一个binlog文件）

b、FROM pos:指定从哪个pos起始点开始查起（不指定就是从整个文件首个pos点开始算）

c、LIMIT【offset】：偏移量\(不指定就是0\)

d、row\_count :查询总条数（不指定就是所有行）

![](/static/image/1414258-20180911102458325-913289497.png)

3、上面这条语句可以将指定的binlog日志文件，分成有效事件行的方式返回，并可使用limit指定pos点的起始偏移，查询条数！

* a、查询第一个最早的binlog日志：

`show binlog events\G;`

* b、指定查询mysql-bin.000002这个文件

`show binlog events in 'mysql-bin.000002'\G;`

* c、指定查询mysql-bin.000002这个文件，从pos点:624开始查起：

`show binlog events in 'mysql-bin.000002' from 624\G;`

* d、指定查询mysql-bin.000002这个文件，从pos点:624开始查起，查询10条（即10条语句）

`show binlog events in 'mysql-bin.000002' from 624 limit 10\G;`

* e、指定查询 mysql-bin.000002这个文件，从pos点:624开始查起，偏移2行（即中间跳过2个）查询10条（即10条语句）。

`show binlog events in 'mysql-bin.000002' from 624 limit 2,10\G;`

## 1.5.利用binlog日志恢复mysql数据

# 2.总结

所谓恢复，就是让mysql将保存在binlog日志中指定段落区间的sql语句逐个重新执行一次而已。

# 3.参考

mysqlbinlog 工具分析binlog日志  
[https://www.cnblogs.com/lvzf/p/10689462.html](https://www.cnblogs.com/lvzf/p/10689462.html)  
mysql binlog详解【重要而且更加详细】  
[https://www.cnblogs.com/Presley-lpc/p/9619571.html](https://www.cnblogs.com/Presley-lpc/p/9619571.html)

