### MVCC

* [《【mysql】关于innodb中MVCC的一些理解》](https://www.cnblogs.com/chenpingzhao/p/5065316.html)

  * innodb 中 MVCC 用在 Repeatable-Read 隔离级别。
  * MVCC 会产生幻读问题（更新时异常。）

* [《轻松理解MYSQL MVCC 实现机制》](https://blog.csdn.net/whoamiyang/article/details/51901888)

  * 通过隐藏版本列来实现 MVCC 控制，一列记录创建时间、一列记录删除时间，这里的时间
  * 每次只操作比当前版本小（或等于）的 行。



