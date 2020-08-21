# 1.MySQL45讲总结

## WAL 技术
WAL 的全称是 Write-Ahead Logging
它的关键点就是先写日志，再写磁盘


## 什么是change buffer？
**《09 | 普通索引和唯一索引，应该怎么选择？》**
当需要更新一个数据页时，如果数据页在内存中就直接更新，而如果这个数据页还没有在内存中的话，在不影响数据一致性的前提下，InnoDB 会将这些更新操作缓存在 change buffer 中，这样就不需要从磁盘中读入这个数据页了。

在下次查询需要访问这个数据页的时候，将数据页读入内存，然后执行 change buffer 中与这个页有关的操作。

通过这种方式就能保证这个数据逻辑的正确性。需要说明的是，虽然名字叫作 change buffer，实际上它是可以持久化的数据。也就是说，change buffer 在内存中有拷贝，也会被写入到磁盘上。将 change buffer 中的操作应用到原数据页，得到最新结果的过程称为 merge。

除了访问这个数据页会触发 merge 外，系统有后台线程会定期 merge。在数据库正常关闭（shutdown）的过程中，也会执行 merge 操作。显然，如果能够将更新操作先记录在 change buffer，减少读磁盘，语句的执行速度会得到明显的提升。而且，数据读入内存是需要占用 buffer pool 的，所以这种方式还能够避免占用内存，提高内存利用率。

change buffer 的大小，可以通过参数 **innodb_change_buffer_max_size** 来动态设置。这个参数设置为 50 的时候，表示 change buffer 的大小最多只能占用 buffer pool 的 50%。

## 什么是buffer pool？
数据库缓冲池(buffer pool)
缓存表数据与索引数据，把磁盘上的数据加载到缓冲池，避免每次访问都进行磁盘IO，起到加速访问的作用。
数据库缓冲池(buffer pool)：`https://www.jianshu.com/p/f9ab1cb24230`