# 1.[MySQL 性能优化神器 Explain 使用分析](https://segmentfault.com/a/1190000008131735) {#sf-article_title}

MySQL 提供了一个 EXPLAIN 命令, 它可以对 SELECT 语句进行分析, 并输出 SELECT 执行的详细信息, 以供开发人员针对性优化.  
EXPLAIN 命令用法十分简单, 在 SELECT 语句前加上 Explain 就可以了, 例如:

```
EXPLAIN SELECT * from user_info WHERE id < 300;
```

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

# 2.参考

MySQL 性能优化神器 Explain 使用分析：

https://segmentfault.com/a/1190000008131735



