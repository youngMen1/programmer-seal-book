## [Java CAS 和ABA问题](https://www.cnblogs.com/549294286/p/3766717.html)

独占锁：是一种悲观锁，synchronized就是一种独占锁，会导致其它所有需要锁的线程挂起，等待持有锁的线程释放锁。

乐观锁：每次不加锁，假设没有冲突去完成某项操作，如果因为冲突失败就重试，直到成功为止。

**一、CAS 操作**

乐观锁用到的机制就是CAS，Compare and Swap。

CAS有3个操作数，内存值V，旧的预期值A，要修改的新值B。当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做。

**1、非阻塞算法 （nonblocking algorithms）**

```
一个线程的失败或者挂起不应该影响其他线程的失败或挂起的算法。
```

现代的CPU提供了特殊的指令，可以自动更新共享数据，而且能够检测到其他线程的干扰，而 compareAndSet\(\) 就用这些代替了锁定。

**2、AtomicInteger示例**

拿出AtomicInteger来研究在没有锁的情况下是如何做到数据正确性的。

```
private volatile int value;
```

在没有锁的机制下需要借助volatile原语，保证线程间的数据是可见的（共享的）。

这样才获取变量的值的时候才能直接读取。

