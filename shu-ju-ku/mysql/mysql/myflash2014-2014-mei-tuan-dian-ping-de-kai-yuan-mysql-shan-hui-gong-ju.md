# 1.MyFlash美团点评的开源MySQL闪回工具

由于运维、DBA的误操作或是业务bug，我们在操作中时不时会出现误删除数据情况。早期要想恢复数据，只能让业务人员根据线上操作日志，构造误删除的数据，或者DBA使用binlog和备份的方式恢复数据，不管那种，都非常费时费力，而且容易出错。直到彭立勋首次在MySQL社区为mysqlbinlog扩展了闪回功能。

在美团点评，我们也遇到过研发人员误删主站的配置信息，从而导致主站长达2个小时不可用的情况。DBA同学当时使用了技术团队自研的binlog2sql完成了数据恢复，并多次挽救了线上误删数据导致的严重故障。不过，binlog2sql在恢复速度上不尽如人意，因此我们开发了一个新的工具——MyFlash，它很好地解决了上述痛点，能够方便并且高效地进行数据恢复。

现在该工具正式开源，开源地址为：https://github.com/Meituan-Dianping/MyFlash

先来看下目前市面上已有的恢复工具，我们从实现角度把它们划分成如下几类。

① mysqlbinlog工具配合sed、awk。该方式先将binlog解析成类SQL的文本，然后使用sed、awk把类SQL文本转换成真正的SQL。 * 优点：当SQL中字段类型比较简单时，可以快速生成需要的SQL，且编程门槛也比较低。 * 缺点：当SQL中字段类型比较复杂时，尤其是字段中的文本包含HTML代码，用awk、sed等工具时，就需要考虑极其复杂的转义等情况，出错概率很大。

② 给数据库源码打patch。该方式扩展了mysqlbinlog的功能，增加Flashback选项。 * 优点：复用了MySQL Server层中binlog解析等代码，一旦稳定之后，无须关心复杂的字段类型，且效率较高。 * 缺点：在修改前，需要对MySQL的复制代码结构和细节需要较深的了解。版本比较敏感，在MySQL 5.6上做的patch，基本不能用于MySQL 5.7的回滚操作。升级困难，因为patch的代码是分布在MySQL的各个文件和函数中，一旦MySQL代码改变，特别是复制层的重构，升级的难度不亚于完全重新写一个。

③ 使用业界提供的解析binlog的库，然后进行SQL构造，其优秀代表是binlog2sql。 * 优点：使用业界成熟的库，因此稳定性较好，且上手难度较低。 * 缺点：效率往往较低，且实现上受制于binlog库提供的功能。

上述几种实现方式，主要是提供的过滤选项较少，比如不能提供基于SQL类型的过滤，需要回滚一个delete语句，导致在回滚时，需要结合awk、sed等工具进行筛选。

总结了上述几种工具的优缺点，我认为理想的闪回工具需要有以下特性。

a. 无需把binlog解析成文本，再进行转换。 b. 提供原生的基于库、表、SQL类型、位置、时间等多种过滤方式。 c. 支持MySQL多个版本。 d. 对于数据库的代码重构不敏感，利于升级。 e. 自主掌控binlog解析，提供尽可能灵活的方式。

**在这些特性中，binlog的解析是一切工作的基础。接下来我会介绍binlog的基本结构。**

## 1.1.binlog格式概览

一个完整的binlog文件是由一个format description event开头，一个rotate event结尾，中间由多个其他event组合而成。

![](/static/image/640a302f.png)

binlog文件实例：

![](/static/image/df3aea56.png)

每个event都是由event header 和event data组成。下面简单介绍下几种常见的binlog event。

① formart description event
![](/static/image/01f23313.png)

表达的含义是：

```
170905  01:59:33 server id 10  end_log_pos 123 CRC32 0xed1ec563 
Start: binlog v 4, server v 5.7.18-log created 170905  01:59:33
```

② table map event

![](/static/image/1ec5b317.png)

表达的含义是：
   
```
170905  01:59:33 server id 10  end_log_pos 339 CRC32 0x3de40c0d     
Table_map: `test`.`test4` mapped to number 238
```

③ update row event
![](/static/image/a39ad60b.png)

# 2.总结

# 3.参考

MyFlash——美团点评的开源MySQL闪回工具

[https://tech.meituan.com/2017/11/17/mysql-flashback.html](https://tech.meituan.com/2017/11/17/mysql-flashback.html)

