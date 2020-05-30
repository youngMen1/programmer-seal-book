# 1.[MySQL 性能优化神器 Explain 使用分析](https://segmentfault.com/a/1190000008131735) {#sf-article_title}

MySQL 提供了一个 EXPLAIN 命令, 它可以对 SELECT 语句进行分析, 并输出 SELECT 执行的详细信息, 以供开发人员针对性优化.  
EXPLAIN 命令用法十分简单, 在 SELECT 语句前加上 Explain 就可以了, 例如:

```
EXPLAIN SELECT * from user_info WHERE id < 300;
EXPLAIN SELECT * FROM `process_info` where id='1e2c4584-0043-487d-86ba-9d2b907e4771';
```

![](/static/image/微信截图_20200530173918.png)

## 1.1.各列的含义如下

* id: SELECT 查询的标识符. 每个 SELECT 都会自动分配一个唯一的标识符.

* select\_type: SELECT 查询的类型.

* table: 查询的是哪个表

* partitions: 匹配的分区

* type: join 类型

* possible\_keys: 此次查询中可能选用的索引

* key: 此次查询中确切使用到的索引.

* ref: 哪个字段或常数与 key 一起被使用

* rows: 显示此查询一共扫描了多少行. 这个是一个估计值.

* filtered: 表示此查询条件所过滤的数据的百分比

* extra: 额外的信息

### select\_type {#item-3-1}

`select_type`表示了查询的类型, 它的常用取值有:

* SIMPLE, 表示此查询不包含 UNION 查询或子查询

* PRIMARY, 表示此查询是最外层的查询

* UNION, 表示此查询是 UNION 的第二或随后的查询

* DEPENDENT UNION, UNION 中的第二个或后面的查询语句, 取决于外面的查询

* UNION RESULT, UNION 的结果

* SUBQUERY, 子查询中的第一个 SELECT

* DEPENDENT SUBQUERY: 子查询中的第一个 SELECT, 取决于外面的查询. 即子查询依赖于外层查询的结果.

# 2.参考

MySQL 性能优化神器 Explain 使用分析：

[https://segmentfault.com/a/1190000008131735](https://segmentfault.com/a/1190000008131735)

