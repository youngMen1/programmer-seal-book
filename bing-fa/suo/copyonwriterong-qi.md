### CopyOnWrite容器

可以对CopyOnWrite容器进行并发的读，而不需要加锁。CopyOnWrite并发容器用于读多写少的并发场景。比如白名单，黑名单，商品类目的访问和更新场景，不适合需要数据强一致性的场景。

* [《JAVA中写时复制\(Copy-On-Write\)Map实现》](https://www.cnblogs.com/hapjin/p/4840107.html)

  * 实现读写分离，读取发生在原始数据上，写入发生在副本上。
  * 不用加锁，通过最终一致实现一致性。

* [《聊聊并发-Java中的Copy-On-Write容器》](https://blog.csdn.net/a494303877/article/details/53404623)



