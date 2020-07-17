# 1.MySQL SYS库常用SQL汇总大全
查看当前连接情况：

```
select host, current_connections,statements from sys.host_summary;
```

查看当前正在执行的SQL：

```
select conn_id, user, current_statement, last_statement from sys.session;
```

查看系统里执行最多的TOP 10 SQL:

```
select * from sys.statement_analysis order by exec_count desc limit 10 \G
```

查看系统里哪张表的IO最多：

```
select * from sys.io_global_by_file_by_bytes limit 10;
```

查看系统里哪张表访问次数最多:

```
select * from sys.statement_analysis order by exec_count desc limit 10 \G
```
查看哪些语句延迟比较严重：

```
select * from sys.statement_analysis order by avg_latency desc limit 10 \G
```

查看系统里未使用过的索引：

```
select * from sys.schema_unused_indexes;
```

查看系统里冗余的索引：

```
select table_schema,table_name,redundant_index_name,redundant_index_columns,dominant_index_name,dominant_index_columns from sys.schema_redundant_indexes;
```

哪些SQL语句使用了磁盘临时表：

```
select db, query, tmp_tables,tmp_disk_tables from sys.statement_analysis where tmp_tables>0 or tmp_disk_tables >0 order by(tmp_tables+tmp_disk_tables) desc limit 20;
```

查看哪张表占用了最多的buffer pool：

```
select * from sys.innodb_buffer_stats_by_table order by pages desc limit 10 \G
```

查看每个库占用多少buffer pool：

```
select * from sys.innodb_buffer_stats_by_schema;
```





查看每个连接分配多少内存:



```
select b.user, current_count_used,current_allocated, current_avg_alloc, current_max_alloc,total_allocated,current_statement from sys.memory_by_thread_by_current_bytes a,sys.session b where a.thread_id = b.thd_id;
```

查看MySQL内部的线程类型及数量：


```
select user, count(*) from sys.processlist group by user;
```

查看表自增ID情况：

```
select * from sys.schema_auto_increment_columns limit 10;
```

# 2.参考
http://blog.itpub.net/15498/viewspace-2643654/

