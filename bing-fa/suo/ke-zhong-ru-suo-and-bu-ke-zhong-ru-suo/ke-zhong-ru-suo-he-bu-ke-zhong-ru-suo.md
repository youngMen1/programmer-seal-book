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



