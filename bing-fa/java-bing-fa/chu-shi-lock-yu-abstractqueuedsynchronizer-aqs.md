## 1. concurrent包的结构层次

在针对并发编程中，Doug Lea大师为我们提供了大量实用，高性能的工具类，针对这些代码进行研究会让我们队并发编程的掌握更加透彻也会大大提升我们队并发编程技术的热爱。这些代码在java.util.concurrent包下。如下图，即为concurrent包的目录结构图。

![](/assets/concurrent目录结构.png)

!\[concurrent目录结构.png\]\([http://upload-images.jianshu.io/upload\_images/2615789-da951eb99c5dabfd.png?imageMogr2/auto-orient/strip\|imageView2/2/w/1240\](http://upload-images.jianshu.io/upload_images/2615789-da951eb99c5dabfd.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240%29\)

其中包含了两个子包：**atomic以及lock**，另外在concurrent下的阻塞队列以及executors,这些就是concurrent包中的精华，之后会一一进行学习。而这些类的实现主要是依赖于volatile以及CAS（关于volatile可以看\[这篇文章\]

\([https://juejin.im/post/5ae9b41b518825670b33e6c4\)，关于CAS可以看\[这篇文章的3.1节\]\(https://juejin.im/post/5ae6dc04f265da0ba351d3ff\)），从整体上来看concurrent包的整体实现图如下图所示：](https://juejin.im/post/5ae9b41b518825670b33e6c4%29，关于CAS可以看[这篇文章的3.1节]%28https://juejin.im/post/5ae6dc04f265da0ba351d3ff%29），从整体上来看concurrent包的整体实现图如下图所示：)

![](/assets/concurrent包实现整体示意图.png)

!\[concurrent包实现整体示意图.png\]\([http://upload-images.jianshu.io/upload\_images/2615789-24da822ddc226b03.png?imageMogr2/auto-orient/strip\|imageView2/2/w/1240\](http://upload-images.jianshu.io/upload_images/2615789-24da822ddc226b03.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240%29\)

## 2. lock简介

我们下来看concurent包下的lock子包。锁是用来控制多个线程访问共享资源的方式，一般来说，一个锁能够防止多个线程同时访问共享资源。在Lock接口出现之前，java程序主要是靠synchronized关键字实现锁功能的，而java SE5之后，并发包中增加了lock接口，它提供了与synchronized一样的锁功能。\*\***虽然它失去了像synchronize关键字隐式加锁解锁的便捷性，但是却拥有了锁获取和释放的可操作性，可中断的获取锁以及超时获取锁等多种synchronized关键字所不具备的同步特性。**\*\*通常使用显示使用lock的形式如下：

```
Lock lock = new ReentrantLock();
    lock.lock();
    try{
        .......
    }finally{
        lock.unlock();
    }
```

需要注意的是\*\***synchronized同步块执行完成或者遇到异常是锁会自动释放，而lock必须调用unlock\(\)方法释放锁，因此在finally块中释放锁**\*\*。

### 2.1 Lock接口API

我们现在就来看看lock接口定义了哪些方法：

&gt; void lock\(\); //获取锁

&gt; void lockInterruptibly\(\) throws InterruptedException；//获取锁的过程能够响应中断

&gt; boolean tryLock\(\);//非阻塞式响应中断能立即返回，获取锁放回true反之返回fasle

&gt; boolean tryLock\(long time, TimeUnit unit\) throws InterruptedException;//超时获取锁，在超时内或者未中断的情况下能够获取锁

&gt; Condition newCondition\(\);//获取与lock绑定的等待通知组件，当前线程必须获得了锁才能进行等待，进行等待时会先释放锁，当再次获取锁时才能从等待中返回

上面是lock接口下的五个方法，也只是从源码中英译中翻译了一遍，感兴趣的可以自己的去看看。那么在locks包下有哪些类实现了该接口了？先从最熟悉的ReentrantLock说起。

&gt; public class ReentrantLock implements \*\*Lock\*\*, java.io.Serializable

很显然ReentrantLock实现了lock接口，接下来我们来仔细研究一下它是怎样实现的。当你查看源码时你会惊讶的发现ReentrantLock并没有多少代码，另外有一个很明显的特点是：\*\*基本上所有的方法的实现实际上都是调用了其静态内存类\`Sync\`中的方法，而Sync类继

承了\`AbstractQueuedSynchronizer（AQS）\`\*\*。可以看出要想理解ReentrantLock关键核心在于对队列同步器AbstractQueuedSynchronizer（简称同步器）的理解。

### 2.2 初识AQS

关于AQS在源码中有十分具体的解释：

```
Provides a framework for implementing blocking locks and related
	 synchronizers (semaphores, events, etc) that rely on
	 first-in-first-out (FIFO) wait queues.  This class is designed to
	 be a useful basis for most kinds of synchronizers that rely on a
	 single atomic {@code int} value to represent state. Subclasses
	 must define the protected methods that change this state, and which
	 define what that state means in terms of this object being acquired
	 or released.  Given these, the other methods in this class carry
	 out all queuing and blocking mechanics. Subclasses can maintain
	 other state fields, but only the atomically updated {@code int}
	 value manipulated using methods {@link #getState}, {@link
	 #setState} and {@link #compareAndSetState} is tracked with respect
	 to synchronization.
	 
	 <p>Subclasses should be defined as non-public internal helper
	 classes that are used to implement the synchronization properties
	 of their enclosing class.  Class
	 {@code AbstractQueuedSynchronizer} does not implement any
	 synchronization interface.  Instead it defines methods such as
	 {@link #acquireInterruptibly} that can be invoked as
	 appropriate by concrete locks and related synchronizers to
	 implement their public methods.
```



