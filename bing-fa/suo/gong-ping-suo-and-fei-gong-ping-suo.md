### 公平锁 & 非公平锁

公平锁的作用就是严格按照线程启动的顺序来执行的，不允许其他线程插队执行的；而非公平锁是允许插队的。

* [《公平锁与非公平锁》](https://blog.csdn.net/EthanWhite/article/details/55508357)
  * 默认情况下 ReentrantLock 和 synchronized 都是非公平锁。ReentrantLock 可以设置成公平锁。



