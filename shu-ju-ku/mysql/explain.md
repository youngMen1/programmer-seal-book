# 1.[MySQL 性能优化神器 Explain 使用分析](https://segmentfault.com/a/1190000008131735) {#sf-article_title}

MySQL 提供了一个 EXPLAIN 命令, 它可以对 SELECT 语句进行分析, 并输出 SELECT 执行的详细信息, 以供开发人员针对性优化.EXPLAIN 命令用法十分简单, 在 SELECT 语句前加上 Explain 就可以了, 例如:

```
EXPLAIN SELECT * from user_info WHERE id < 300;
 
```



