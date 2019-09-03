### 可重入锁 & 不可重入锁

* [《可重入锁和不可重入锁》](https://www.cnblogs.com/dj3839/p/6580765.html)

  * 通过简单代码举例说明可重入锁和不可重入锁。
  * 可重入锁指同一个线程可以再次获得之前已经获得的锁。
  * 可重入锁可以用户避免死锁。
  * Java中的可重入锁：synchronized 和 java.util.concurrent.locks.ReentrantLock

* [《ReenTrantLock可重入锁（和synchronized的区别）总结》](https://www.cnblogs.com/baizhanshi/p/7211802.html)

  * synchronized 使用方便，编译器来加锁，是非公平锁。
  * ReenTrantLock 使用灵活，锁的公平性可以定制。
  * 相同加锁场景下，推荐使用 synchronized。



