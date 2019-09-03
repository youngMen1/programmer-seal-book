### 互斥锁 & 共享锁

互斥锁：同时只能有一个线程获得锁。比如，ReentrantLock 是互斥锁，ReadWriteLock 中的写锁是互斥锁。 共享锁：可以有多个线程同时或的锁。比如，Semaphore、CountDownLatch 是共享锁，ReadWriteLock 中的读锁是共享锁。

* [《ReadWriteLock场景应用》](https://www.cnblogs.com/liang1101/p/6475555.html)



