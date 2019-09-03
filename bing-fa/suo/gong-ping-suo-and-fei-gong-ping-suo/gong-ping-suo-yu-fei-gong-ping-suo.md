# 公平锁与非公平锁

在ReentrantLock中很明显可以看到其中同步包括两种，分别是公平的FairSync和非公平的NonfairSync。公平锁的作用就是严格按照线程启动的顺序来执行的，不允许其他线程插队执行的；而非公平锁是允许插队的。

默认情况下ReentrantLock是通过非公平锁来进行同步的，包括synchronized关键字都是如此，因为这样性能会更好。因为从线程进入了RUNNABLE状态，可以执行开始，到实际线程执行是要比较久的时间的。而且，在一个锁释放之后，其他的线程会需要重新来获取锁。其中经历了持有锁的线程释放锁，其他线程从挂起恢复到RUNNABLE状态，其他线程请求锁，获得锁，线程执行，这一系列步骤。如果这个时候，存在一个线程直接请求锁，可能就避开挂起到恢复RUNNABLE状态的这段消耗，所以性能更优化。

```
 /**
     * Creates an instance of {@code ReentrantLock}.
     * This is equivalent to using {@code ReentrantLock(false)}.
     */
    public ReentrantLock() {
        sync = new NonfairSync();
    }
————————————————
版权声明：本文为CSDN博主「EthanPark」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/EthanWhite/article/details/55508357
```



