## 锁的简单应用

用lock来保证原子性（this.count++这段代码称为临界区）

什么是原子性，就是不可分，从头执行到尾，不能被其他线程同时执行。

可通过CAS来实现原子操作

CAS\(Compare and Swap\):

CAS操作需要输入两个数值，一个旧值（期望操作前的值）和一个新值，在操作期间先比较下旧值有没有发生变化，如果没有发生变化，才交换成新值，发生了变化则不交换。

CAS主要通过compareAndSwapXXX\(\)方法来实现，而这个方法的实现需要涉及底层的unsafe类

unsafe类：java不能直接访问操作系统底层，而是通过本地方法来访问。Unsafe类提供了硬件级别的原子操作

这里有个介绍原子操作的博客

[https://my.oschina.net/xinxingegeya/blog/499223](https://my.oschina.net/xinxingegeya/blog/499223)

还有对unsafe类详解的博客

[http://www.cnblogs.com/mickole/articles/3757278.html](http://www.cnblogs.com/mickole/articles/3757278.html)

```
public class Counter{
    private Lock lock = new Lock();
    private int count = 0;
    public int inc(){
        lock.lock();
        this.count++;
        lock.unlock();
        return count;
    }
}
```

## 不可重入锁

```
public class Lock{
    private boolean isLocked = false;
    public synchronized void lock() throws InterruptedException{
        while(isLocked){    
            wait();
        }
        isLocked = true;
    }
    public synchronized void unlock(){
        isLocked = false;
        notify();
    }
}
```

这其实是个不可重入锁，举个例子

```
public class Count{
    Lock lock = new Lock();
    public void print(){
        lock.lock();
        doAdd();
        lock.unlock();
    }
    public void doAdd(){
        lock.lock();
        //do something
        lock.unlock();
    }
}
```

当调用print\(\)方法时，获得了锁，这时就无法再调用doAdd\(\)方法，这时必须先释放锁才能调用，所以称这种锁为不可重入锁，也叫自旋锁。

## 可重入锁

```
public class Lock{
    boolean isLocked = false;
    Thread  lockedBy = null;
    int lockedCount = 0;
    public synchronized void lock()
            throws InterruptedException{
        Thread thread = Thread.currentThread();
        while(isLocked && lockedBy != thread){
            wait();
        }
        isLocked = true;
        lockedCount++;
        lockedBy = thread;
    }
    public synchronized void unlock(){
        if(Thread.currentThread() == this.lockedBy){
            lockedCount--;
            if(lockedCount == 0){
                isLocked = false;
                notify();
            }
        }
    }
}
```

相对来说，可重入就意味着：线程可以进入任何一个它已经拥有的锁所同步着的代码块。

第一个线程执行print\(\)方法，得到了锁，使lockedBy等于当前线程，也就是说，执行的这个方法的线程获得了这个锁，执行add\(\)方法时，同样要先获得锁，因不满足while循环的条件，也就是不等待，继续进行，将此时的lockedCount变量，也就是当前获得锁的数量加一，当释放了所有的锁，才执行notify\(\)。如果在执行这个方法时，有第二个线程想要执行这个方法，因为lockedBy不等于第二个线程，导致这个线程进入了循环，也就是等待，不断执行wait\(\)方法。只有当第一个线程释放了所有的锁，执行了notify\(\)方法，第二个线程才得以跳出循环，继续执行。

这就是可重入锁的特点。

java中常用的可重入锁

synchronized

java.util.concurrent.locks.ReentrantLock

ps:顺便记录下java中实现原子操作的类（记录至[http://blog.csdn.net/huzhigenlaohu/article/details/51646455](http://blog.csdn.net/huzhigenlaohu/article/details/51646455)）

* AtomicIntegerFieldUpdater:原子更新整型的字段的更新器
* AtomicLongFieldUpdater：原子更新长整型字段的更新器
* AtomicStampedReference:原子更新带有版本号的引用类型。该类将整型数值与引用关联起来，可用于原子的更新数据和数据的版本号，可以解决使用CAS进行原子更新时可能出现的ABA问题。

* AtomicReference ：原子更新引用类型
* AtomicReferenceFieldUpdater ：原子更新引用类型里的字段
* AtomicMarkableReference：原子更新带有标记位的引用类型。可以原子更新一个布尔类型的标记位和应用类型

* AtomicIntegerArray ：原子更新整型数组里的元素
* AtomicLongArray :原子更新长整型数组里的元素
* AtomicReferenceArray : 原子更新引用类型数组的元素
* AtomicBooleanArray ：原子更新布尔类型数组的元素

* AtomicBoolean ：原子更新布尔类型
* AtomicInteger： 原子更新整型
* AtomicLong: 原子更新长整型



