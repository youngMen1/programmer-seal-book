# [并发库应用之五 & ReadWriteLock场景应用](https://www.cnblogs.com/liang1101/p/6475555.html)

Lock比传统线程模型中的synchronized方式更加面向对象，与生活中的锁类似，锁本身也应该是一个对象。

两个线程执行的代码片段要实现同步互斥的效果，它们必须用同一个Lock对象。

读写锁：分为读锁和写锁，多个读锁不互斥，读锁与写锁互斥，这是由jvm自己控制的，我们只要上好相应的锁即可。

如果你的代码只读数据，可以很多人同时读，但不能同时写，那就上读锁；如果你的代码修改数据，只能有一个人在写，且不能同时读取，那就上写锁。总之，读的时候上读锁，写的时候上写锁！

**读写锁接口：ReadWriteLock，它的具体实现类为：ReentrantReadWriteLock**

```
在多线程的环境下，对同一份数据进行读写，会涉及到线程安全的问题。比如在一个线程读取数据的时候，
另外一个线程在写数据，而导致前后数据的不一致性；一个线程在写数据的时候，另一个线程也在写，
同样也会导致线程前后看到的数据的不一致性。

这时候可以在读写方法中加入互斥锁，任何时候只能允许一个线程的一个读或写操作，而不允许其他线程的读或写操作，
这样是可以解决这样以上的问题，但是效率却大打折扣了。因为在真实的业务场景中，
一份数据，读取数据的操作次数通常高于写入数据的操作，而线程与线程间的读读操作是不涉及到线程安全的问题，
没有必要加入互斥锁，只要在读-写，写-写期间上锁就行了。
```

对于以上这种情况，读写锁是最好的解决方案！其中它的实现类：ReentrantReadWriteLock--顾名思义是可重入的读写锁，允许多个读线程获得ReadLock，但只允许一个写线程获得WriteLock

读写锁的机制：

"读-读" 不互斥

"读-写" 互斥

"写-写" 互斥

ReentrantReadWriteLock会使用两把锁来解决问题，一个读锁，一个写锁。

**线程进入读锁的前提条件：**

1. 没有其他线程的写锁

2. 没有写请求，或者有写请求但调用线程和持有锁的线程是同一个线程

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

以上这段代码虽然不会导致死锁，但没有正确的释放锁。从写锁降级成读锁，并不会自动释放当前线程获取的写锁，仍然需要显示的释放，否则别的线程永远也获取不到写锁。

---

以下我会通过一个真实场景下的缓存机制来讲解 ReentrantReadWriteLock 实际应用

---

首先来看看

ReentrantReadWriteLock的javaodoc

文档中提供给我们的一个很好的Cache实例代码案例：

```
class CachedData {
  Object data;
  volatile boolean cacheValid;
  final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();

  public void processCachedData() {
    rwl.readLock().lock();
    if (!cacheValid) {
      // Must release read lock before acquiring write lock
      rwl.readLock().unlock();
      rwl.writeLock().lock();
      try {
        // Recheck state because another thread might have,acquired write lock and changed state before we did.
        if (!cacheValid) {
          data = ...
          cacheValid = true;
        }
        // 在释放写锁之前通过获取读锁降级写锁(注意此时还没有释放写锁)
        rwl.readLock().lock();
      } finally {
        rwl.writeLock().unlock(); // 释放写锁而此时已经持有读锁
      }
    }

    try {
      use(data);
    } finally {
      rwl.readLock().unlock();
    }
  }
}
```

以上代码加锁的顺序为：

**1.**rwl.readLock\(\).lock\(\);

**2.**rwl.readLock\(\).unlock\(\);

**3.**rwl.writeLock\(\).lock\(\);

**4.**rwl.readLock\(\).lock\(\);

**5.**rwl.writeLock\(\).unlock\(\);

**6.**rwl.readLock\(\).unlock\(\);

以上过程整体讲解：

1. 多个线程同时访问该缓存对象时，都加上当前对象的读锁，之后其中某个线程优先查看data数据是否为空。【加锁顺序序号：1 】

2. 当前查看的线程发现没有值则释放读锁立即加上写锁，准备写入缓存数据。（不明白为什么释放读锁的话可以查看上面讲解进入写锁的前提条件）【加锁顺序序号：2和3 】

3. 为什么还会再次判断是否为空值（!cacheValid）是因为第二个、第三个线程获得读的权利时也是需要判断是否为空，否则会重复写入数据。

4. 写入数据后先进行读锁的降级后再释放写锁。【加锁顺序序号：4和5 】

5. 最后数据数据返回前释放最终的读锁。【加锁顺序序号：6 】

如果不使用锁降级功能，如先释放写锁，然后获得读锁，在这个get过程中，可能会有其他线程竞争到写锁 或者是更新数据 则获得的数据是其他线程更新的数据，可能会造成数据的污染，即产生脏读的问题。

下面，让我们来实现真正趋于实际生产环境中的缓存案例：

```
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class CacheDemo {
    /**
     * 缓存器,这里假设需要存储1000左右个缓存对象，按照默认的负载因子0.75，则容量=750，大概估计每一个节点链表长度为5个
     * 那么数组长度大概为：150,又有雨设置map大小一般为2的指数，则最近的数字为：128
     */
    private Map<String, Object> map = new HashMap<>(128);
    private ReadWriteLock rwl = new ReentrantReadWriteLock();
    public static void main(String[] args) {

    }
    public Object get(String id){
        Object value = null;
        rwl.readLock().lock();//首先开启读锁，从缓存中去取
        try{
               if(map.get(id) == null){  //如果缓存中没有释放读锁，上写锁
                rwl.readLock().unlock();
                rwl.writeLock().lock();
                try{
                    if(value == null){ //防止多写线程重复查询赋值
                        value = "redis-value";  //此时可以去数据库中查找，这里简单的模拟一下
                    }
                    rwl.readLock().lock(); //加读锁降级写锁,不明白的可以查看上面锁降级的原理与保持读取数据原子性的讲解
                }finally{
                    rwl.writeLock().unlock(); //释放写锁
                }
            }
        }finally{
            rwl.readLock().unlock(); //最后释放读锁
        }
        return value;
    }
}
```

**提示：**

读写锁之后有一个与它配合使用的有条件的阻塞，可以实现线程间的通信，它就是Condition。具体详情请查看我的博客：

[并发库应用之六 &有条件阻塞Condition应用](http://www.cnblogs.com/liang1101/p/6522375.html)

