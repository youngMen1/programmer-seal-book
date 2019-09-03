# [并发库应用之五 & ReadWriteLock场景应用](https://www.cnblogs.com/liang1101/p/6475555.html)

Lock比传统线程模型中的synchronized方式更加面向对象，与生活中的锁类似，锁本身也应该是一个对象。

两个线程执行的代码片段要实现同步互斥的效果，它们必须用同一个Lock对象。

读写锁：分为读锁和写锁，多个读锁不互斥，读锁与写锁互斥，这是由jvm自己控制的，我们只要上好相应的锁即可。如果你的代码只读数据，可以很多人同时读，但不能同时写，那就上读锁；如果你的代码修改数据，只能有一个人在写，且不能同时读取，那就上写锁。总之，读的时候上读锁，写的时候上写锁！

**读写锁接口**

**：ReadWriteLock，它的具体实现类为：ReentrantReadWriteLock**

```
   在多线程的环境下，对同一份数据进行读写，会涉及到线程安全的问题。比如在一个线程读取数据的时候，另外一个线程在写数据，而导致前后数据的不一致性；一个线程在写数据的时候，另一个线程也在写，同样也会导致线程前后看到的数据的不一致性。

   这时候可以在读写方法中加入互斥锁，任何时候只能允许一个线程的一个读或写操作，而不允许其他线程的读或写操作，这样是可以解决这样以上的问题，但是效率却大打折扣了。因为在真实的业务场景中，一份数据，读取数据的操作次数通常高于写入数据的操作，而线程与线程间的读读操作是不涉及到线程安全的问题，没有必要加入互斥锁，只要在读-写，写-写期间上锁就行了。
```

对于以上这种情况，读写锁是最好的解决方案！其中它的实现类：ReentrantReadWriteLock--顾名思义是可重入的读写锁，允许多个读线程获得ReadLock，但只允许一个写线程获得WriteLock

读写锁的机制：

"读-读" 不互斥

"读-写" 互斥

"写-写" 互斥

ReentrantReadWriteLock会使用两把锁来解决问题，一个读锁，一个写锁。

**线程进入读锁的前提条件：**

```
 　　 1. 没有其他线程的写锁
```

1. 没有写请求，或者有写请求但调用线程和持有锁的线程是同一个线程

**进入写锁的前提条件：**

1. 没有其他线程的读锁

2. 没有其他线程的写锁

**需要提前了解的概念：**

锁降级：从写锁变成读锁；

锁升级：从读锁变成写锁。

读锁是可以被多线程共享的，写锁是单线程独占的。也就是说写锁的并发限制比读锁高，这可能就是升级/降级名称的来源。

如下代码会产生死锁，因为同一个线程中，在没有释放读锁的情况下，就去申请写锁，这属于**锁升级，ReentrantReadWriteLock是不支持**的。

```
 ReadWriteLock rtLock = new ReentrantReadWriteLock();
 rtLock.readLock().lock();
 System.out.println("get readLock.");
 rtLock.writeLock().lock();
 System.out.println("blocking");
```

**ReentrantReadWriteLock支持锁降级，**

如下代码不会产生死锁。

```
ReadWriteLock rtLock = new ReentrantReadWriteLock();
rtLock.writeLock().lock();
System.out.println("writeLock");

rtLock.readLock().lock();
System.out.println("get read lock");
```



