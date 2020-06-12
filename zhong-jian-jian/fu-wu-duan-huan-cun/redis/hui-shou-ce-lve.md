# Redis的回收策略

## 1.使用[Redis](http://lib.csdn.net/base/redis)有哪些好处？

\(1\) 速度快，因为数据存在内存中，类似于HashMap，HashMap的优势就是查找和操作的时间复杂度都是O\(1\)

\(2\) 支持丰富数据类型，支持string，list，set，sorted set，hash

\(3\) 支持事务，操作都是原子性，所谓的原子性就是对数据的更改要么全部执行，要么全部不执行

\(4\) 丰富的特性：可用于缓存，消息，按key设置过期时间，过期后将会自动删除

## 2.redis相比memcached有哪些优势？

\(1\) memcached所有的值均是简单的字符串，redis作为其替代者，支持更为丰富的数据类型

\(2\) redis的速度比memcached快很多

\(3\) redis可以持久化其数据

## 3.redis常见性能问题和解决方案：

\(1\) Master最好不要做任何持久化工作，如RDB内存快照和AOF日志文件

\(2\) 如果数据比较重要，某个Slave开启AOF备份数据，策略设置为每秒同步一次

\(3\) 为了主从复制的速度和连接的稳定性，Master和Slave最好在同一个局域网内

\(4\) 尽量避免在压力很大的主库上增加从库

\(5\) 主从复制不要用图状结构，用单向链表结构更为稳定，即：Master &lt;- Slave1 &lt;- Slave2 &lt;- Slave3...

这样的结构方便解决单点故障问题，实现Slave对Master的替换。如果Master挂了，可以立刻启用Slave1做Master，其他不变。

## 4.Redis的回收策略

* volatile-lru：从已设置过期时间的数据集（server.db\[i\].expires）中挑选最近最少使用的数据淘汰

* volatile-ttl：从已设置过期时间的数据集（server.db\[i\].expires）中挑选将要过期的数据淘汰

* volatile-random：从已设置过期时间的数据集（server.db\[i\].expires）中任意选择数据淘汰

* allkeys-lru：从数据集（server.db\[i\].dict）中挑选最近最少使用的数据淘汰

* allkeys-random：从数据集（server.db\[i\].dict）中任意选择数据淘汰

* no-enviction（驱逐）：禁止驱逐数据



