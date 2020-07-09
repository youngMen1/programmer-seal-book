# 1.MySQL为什么有时候会选错索引？

前面我们介绍过索引，你已经知道了在 MySQL 中一张表其实是可以支持多个索引的。但是，你写 SQL 语句的时候，并没有主动指定使用哪个索引。也就是说，使用哪个索引是由 MySQL 来确定的。

不知道你有没有碰到过这种情况，一条本来可以执行得很快的语句，却由于 MySQL 选错了索引，而导致执行速度变得很慢？

我们一起来看一个例子吧。

我们先建一个简单的表，表里有 a、b 两个字段，并分别建上索引：

    CREATE TABLE `t` (
      `id` int(11) NOT NULL,
      `a` int(11) DEFAULT NULL,
      `b` int(11) DEFAULT NULL,
      PRIMARY KEY (`id`),
      KEY `a` (`a`),
      KEY `b` (`b`)
    ) ENGINE=InnoDB；

然后，我们往表 t 中插入 10 万行记录，取值按整数递增，即：\(1,1,1\)，\(2,2,2\)，\(3,3,3\) 直到 \(100000,100000,100000\)。

我是用存储过程来插入数据的，这里我贴出来方便你复现：

```
delimiter ;;
create procedure idata()
begin
  declare i int;
  set i=1;
  while(i<=100000)do
    insert into t values(i, i, i);
    set i=i+1;
  end while;
end;;
delimiter ;
call idata();
```

接下来，我们分析一条 SQL 语句：

```

mysql> select * from t where a between 10000 and 20000;
```
你一定会说，这个语句还用分析吗，很简单呀，a 上有索引，肯定是要使用索引 a 的。

你说得没错，图 1 显示的就是使用 explain 命令看到的这条语句的执行情况。

![](/static/image/2cfce769551c6eac9bfbee0563d48fe3.png)
                                                                                               图 1 使用 explain 命令查看语句执行情况
                                                                                               
从图 1 看上去，这条查询语句的执行也确实符合预期，key 这个字段值是’a’，表示优化器选择了索引 a。
不过别急，这个案例不会这么简单。在我们已经准备好的包含了 10 万行数据的表上，我们再做如下操作。

![](/static/image/1e5ba1c2934d3b2c0d96b210a27e1a1e.png)
                                                                                              图 2 session A 和 session B 的执行流程
这里，session A 的操作你已经很熟悉了，它就是开启了一个事务。随后，session B 把数据都删除后，又调用了 idata 这个存储过程，插入了 10 万行数据。

这时候，session B 的查询语句 select * from t where a between 10000 and 20000 就不会再选择索引 a 了。我们可以通过慢查询日志（slow log）来查看一下具体的执行情况。
                                                                                                                                                                                            
                                                                                              
# 2.总结



