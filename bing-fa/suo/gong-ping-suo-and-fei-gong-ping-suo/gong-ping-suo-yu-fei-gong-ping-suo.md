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
```

默认状态，使用的ReentrantLock\(\)就是非公平锁。再参考如下代码，我们知道ReentrantLock的获取锁的操作是通过装饰模式代理给sync的。

```
/**
     * Acquires the lock.
     *
     * <p>Acquires the lock if it is not held by another thread and returns
     * immediately, setting the lock hold count to one.
     *
     * <p>If the current thread already holds the lock then the hold
     * count is incremented by one and the method returns immediately.
     *
     * <p>If the lock is held by another thread then the
     * current thread becomes disabled for thread scheduling
     * purposes and lies dormant until the lock has been acquired,
     * at which time the lock hold count is set to one.
     */
    public void lock() {
        sync.lock();
    }
```

下面参考一下FairSync和NonfairSync对lock方法的实现：

```
 /**
     * Sync object for non-fair locks
     */
    static final class NonfairSync extends Sync {
        /**
         * Performs lock.  Try immediate barge, backing up to normal
         * acquire on failure.
         */
        final void lock() {
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
        }
    }

    /**
     * Sync object for fair locks
     */
    static final class FairSync extends Sync {
        final void lock() {
            acquire(1);
        }
    }
```

当使用非公平锁的时候，会立刻尝试配置状态，成功了就会插队执行，失败了就会和公平锁的机制一样，调用acquire\(\)方法，以排他的方式来获取锁，成功了立刻返回，否则将线程加入队列，知道成功调用为止。

