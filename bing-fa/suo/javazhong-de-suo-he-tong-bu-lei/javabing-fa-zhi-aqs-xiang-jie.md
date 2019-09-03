# [Java并发之AQS详解](https://www.cnblogs.com/waterystone/p/4920797.html)

# 一、概述

谈到并发，不得不谈ReentrantLock；而谈到ReentrantLock，不得不谈AbstractQueuedSynchronizer（AQS）！

类如其名，抽象的队列式的同步器，AQS定义了一套多线程访问共享资源的同步器框架，许多同步类实现都依赖于它，如常用的ReentrantLock/Semaphore/CountDownLatch...。

以下是本文的目录大纲：

1. 1. 概述
   2. 框架
   3. 源码详解
   4. 简单应用

若有不正之处，请谅解和批评指正，不胜感激。

请尊重作者劳动成果，转载请标明原文链接：[http://www.cnblogs.com/waterystone/p/4920797.html](http://www.cnblogs.com/waterystone/p/4920797.html)

手机版可访问：[https://mp.weixin.qq.com/s/eyZyzk8ZzjwzZYN4a4H5YA](https://mp.weixin.qq.com/s/eyZyzk8ZzjwzZYN4a4H5YA)

# 二、框架

721070-20170504110246211-10684485.png

它维护了一个volatile int state（代表共享资源）和一个FIFO线程等待队列（多线程争用资源被阻塞时会进入此队列）。这里volatile是核心关键词，具体volatile的语义，在此不述。state的访问方式有三种:

* getState\(\)
* setState\(\)
* compareAndSetState\(\)

AQS定义两种资源共享方式：Exclusive（独占，只有一个线程能执行，如ReentrantLock）和Share（共享，多个线程可同时执行，如Semaphore/CountDownLatch）。

不同的自定义同步器争用共享资源的方式也不同。**自定义同步器在实现时只需要实现共享资源state的获取与释放方式即可**，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在顶层实现好了。自定义同步器实现时主要实现以下几种方法：

* isHeldExclusively\(\)：该线程是否正在独占资源。只有用到condition才需要去实现它。
* tryAcquire\(int\)：独占方式。尝试获取资源，成功则返回true，失败则返回false。
* tryRelease\(int\)：独占方式。尝试释放资源，成功则返回true，失败则返回false。
* tryAcquireShared\(int\)：共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
* tryReleaseShared\(int\)：共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false。

以ReentrantLock为例，state初始化为0，表示未锁定状态。A线程lock\(\)时，会调用tryAcquire\(\)独占该锁并将state+1。此后，其他线程再tryAcquire\(\)时就会失败，直到A线程unlock\(\)到state=0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A线程自己是可以重复获取此锁的（state会累加），这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证state是能回到零态的。

再以CountDownLatch以例，任务分为N个子线程去执行，state也初始化为N（注意N要与线程个数一致）。这N个子线程是并行执行的，每个子线程执行完后countDown\(\)一次，state会CAS减1。等到所有子线程都执行完后\(即state=0\)，会unpark\(\)主调用线程，然后主调用线程就会从await\(\)函数返回，继续后余动作。

一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现tryAcquire-tryRelease、tryAcquireShared-tryReleaseShared中的一种即可。但AQS也支持自定义同步器同时实现独占和共享两种方式，如ReentrantReadWriteLock。

# 三、源码详解

本节开始讲解AQS的源码实现。依照acquire-release、acquireShared-releaseShared的次序来。

## 3.1 acquire\(int\)

此方法是独占模式下线程获取共享资源的顶层入口。如果获取到资源，线程直接返回，否则进入等待队列，直到获取到资源为止，且整个过程忽略中断的影响。这也正是lock\(\)的语义，当然不仅仅只限于lock\(\)。获取到资源后，线程就可以去执行其临界区代码了。下面是acquire\(\)的源码：

```
public final void acquire(int arg) {
     if (!tryAcquire(arg) &&
         acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
         selfInterrupt();
 }
```

函数流程如下：

1. 1. tryAcquire\(\)尝试直接去获取资源，如果成功则直接返回；
   2. addWaiter\(\)将该线程加入等待队列的尾部，并标记为独占模式；
   3. acquireQueued\(\)使线程在等待队列中获取资源，一直获取到资源后才返回。如果在整个等待过程中被中断过，则返回true，否则返回false。
   4. 如果线程在等待过程中被中断过，它是不响应的。只是获取资源后才再进行自我中断selfInterrupt\(\)，将中断补上。

这时单凭这4个抽象的函数来看流程还有点朦胧，不要紧，看完接下来的分析后，你就会明白了。就像《大话西游》里唐僧说的：等你明白了舍生取义的道理，你自然会回来和我唱这首歌的。

### 3.1.1 tryAcquire\(int\)

此方法尝试去获取独占资源。如果获取成功，则直接返回true，否则直接返回false。这也正是tryLock\(\)的语义，还是那句话，当然不仅仅只限于tryLock\(\)。如下是tryAcquire\(\)的源码：

```
protected boolean tryAcquire(int arg) {
         throw new UnsupportedOperationException();
     }
```

什么？直接throw异常？说好的功能呢？好吧，**还记得概述里讲的AQS只是一个框架，具体资源的获取/释放方式交由自定义同步器去实现吗？**就是这里了！！！AQS这里只定义了一个接口，具体资源的获取交由自定义同步器去实现了（通过state的get/set/CAS）！！！至于能不能重入，能不能加塞，那就看具体的自定义同步器怎么去设计了！！！当然，自定义同步器在进行资源访问时要考虑线程安全的影响。

这里之所以没有定义成abstract，是因为独占模式下只用实现tryAcquire-tryRelease，而共享模式下只用实现tryAcquireShared-tryReleaseShared。如果都定义成abstract，那么每个模式也要去实现另一模式下的接口。说到底，Doug Lea还是站在咱们开发者的角度，尽量减少不必要的工作量。

### 3.1.2 addWaiter\(Node\)

此方法用于将当前线程加入到等待队列的队尾，并返回当前线程所在的结点。还是上源码吧：

```
private Node addWaiter(Node mode) {
    //以给定模式构造结点。mode有两种：EXCLUSIVE（独占）和SHARED（共享）
    Node node = new Node(Thread.currentThread(), mode);

    //尝试快速方式直接放到队尾。
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }

    //上一步失败则通过enq入队。
    enq(node);
    return node;
}
```

不用再说了，直接看注释吧。这里我们说下Node。Node结点是对每一个访问同步代码的线程的封装，其包含了需要同步的线程本身以及线程的状态，如是否被阻塞，是否等待唤醒，是否已经被取消等。变量waitStatus则表示当前被封装成Node结点的等待状态，共有4种取值CANCELLED、SIGNAL、CONDITION、PROPAGATE。

* CANCELLED：值为1，在同步队列中等待的线程等待超时或被中断，需要从同步队列中取消该Node的结点，其结点的waitStatus为CANCELLED，即结束状态，进入该状态后的结点将不会再变化。

* SIGNAL：值为-1，被标识为该等待唤醒状态的后继结点，当其前继结点的线程释放了同步锁或被取消，将会通知该后继结点的线程执行。说白了，就是处于唤醒状态，只要前继结点释放锁，就会通知标识为SIGNAL状态的后继结点的线程执行。

* CONDITION：值为-2，与Condition相关，该标识的结点处于**等待队列**中，结点的线程等待在Condition上，当其他线程调用了Condition的signal\(\)方法后，CONDITION状态的结点将**从等待队列转移到同步队列中**，等待获取同步锁。

* PROPAGATE：值为-3，与共享模式相关，在共享模式中，该状态标识结点的线程处于可运行状态。

* 0状态：值为0，代表初始化状态。

AQS在判断状态时，通过用waitStatus&gt;0表示取消状态，而waitStatus&lt;0表示有效状态。

#### 3.1.2.1 enq\(Node\)

此方法用于将node加入队尾。源码如下：

```
private Node enq(final Node node) {
    //CAS"自旋"，直到成功加入队尾
    for (;;) {
        Node t = tail;
        if (t == null) { // 队列为空，创建一个空的标志结点作为head结点，并将tail也指向它。
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {//正常流程，放入队尾
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

如果你看过AtomicInteger.getAndIncrement\(\)函数源码，那么相信你一眼便看出这段代码的精华。**CAS自旋volatile变量**，是一种很经典的用法。还不太了解的，自己去百度一下吧。

### 3.1.3 acquireQueued\(Node, int\)

OK，通过tryAcquire\(\)和addWaiter\(\)，该线程获取资源失败，已经被放入等待队列尾部了。聪明的你立刻应该能想到该线程下一部该干什么了吧：**进入等待状态休息，直到其他线程彻底释放资源后唤醒自己，自己再拿到资源，然后就可以去干自己想干的事了**。没错，就是这样！是不是跟医院排队拿号有点相似~~acquireQueued\(\)就是干这件事：**在等待队列中排队拿号（中间没其它事干可以休息），直到拿到号后再返回**。这个函数非常关键，还是上源码吧：

```
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;//标记是否成功拿到资源
    try {
        boolean interrupted = false;//标记等待过程中是否被中断过

        //又是一个“自旋”！
        for (;;) {
            final Node p = node.predecessor();//拿到前驱
            //如果前驱是head，即该结点已成老二，那么便有资格去尝试获取资源（可能是老大释放完资源唤醒自己的，当然也可能被interrupt了）。
            if (p == head && tryAcquire(arg)) {
                setHead(node);//拿到资源后，将head指向该结点。所以head所指的标杆结点，就是当前获取到资源的那个结点或null。
                p.next = null; // setHead中node.prev已置为null，此处再将head.next置为null，就是为了方便GC回收以前的head结点。也就意味着之前拿完资源的结点出队了！
                failed = false;
                return interrupted;//返回等待过程中是否被中断过
            }

            //如果自己可以休息了，就进入waiting状态，直到被unpark()
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;//如果等待过程中被中断过，哪怕只有那么一次，就将interrupted标记为true
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

到这里了，我们先不急着总结acquireQueued\(\)的函数流程，先看看shouldParkAfterFailedAcquire\(\)和parkAndCheckInterrupt\(\)具体干些什么。

#### 3.1.3.1 shouldParkAfterFailedAcquire\(Node, Node\)

此方法主要用于检查状态，看看自己是否真的可以去休息了（进入waiting状态，如果线程状态转换不熟，可以参考本人上一篇写的[Thread详解](http://www.cnblogs.com/waterystone/p/4920007.html)），万一队列前边的线程都放弃了只是瞎站着，那也说不定，对吧！

```
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;//拿到前驱的状态
    if (ws == Node.SIGNAL)
        //如果已经告诉前驱拿完号后通知自己一下，那就可以安心休息了
        return true;
    if (ws > 0) {
        /*
         * 如果前驱放弃了，那就一直往前找，直到找到最近一个正常等待的状态，并排在它的后边。
         * 注意：那些放弃的结点，由于被自己“加塞”到它们前边，它们相当于形成一个无引用链，稍后就会被保安大叔赶走了(GC回收)！
         */
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
         //如果前驱正常，那就把前驱的状态设置成SIGNAL，告诉它拿完号后通知自己一下。有可能失败，人家说不定刚刚释放完呢！
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```

整个流程中，如果前驱结点的状态不是SIGNAL，那么自己就不能安心去休息，需要去找个安心的休息点，同时可以再尝试下看有没有机会轮到自己拿号。

#### 3.1.3.2 parkAndCheckInterrupt\(\)

如果线程找好安全休息点后，那就可以安心去休息了。此方法就是让线程去休息，真正进入等待状态。

```
private final boolean parkAndCheckInterrupt() {
     LockSupport.park(this);//调用park()使线程进入waiting状态
     return Thread.interrupted();//如果被唤醒，查看自己是不是被中断的。
 }
```

park\(\)会让当前线程进入waiting状态。在此状态下，有两种途径可以唤醒该线程：1）被unpark\(\)；2）被interrupt\(\)。（再说一句，如果线程状态转换不熟，可以参考本人写的[Thread详解](http://www.cnblogs.com/waterystone/p/4920007.html)）。需要注意的是，Thread.interrupted\(\)会清除当前线程的中断标记位。

#### 3.1.3.3 小结

OK，看了shouldParkAfterFailedAcquire\(\)和parkAndCheckInterrupt\(\)，现在让我们再回到acquireQueued\(\)，总结下该函数的具体流程：

1. 结点进入队尾后，检查状态，找到安全休息点；
2. 调用park\(\)进入waiting状态，等待unpark\(\)或interrupt\(\)唤醒自己；
3. 被唤醒后，看自己是不是有资格能拿到号。如果拿到，head指向当前结点，并返回从入队到拿到号的整个过程中是否被中断过；如果没拿到，继续流程1。

### 3.1.4 小结

OKOK，acquireQueued\(\)分析完之后，我们接下来再回到acquire\(\)！再贴上它的源码吧：

```
public final void acquire(int arg) {
     if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
       selfInterrupt();
}
```

再来总结下它的流程吧：

1. 调用自定义同步器的tryAcquire\(\)尝试直接去获取资源，如果成功则直接返回；
2. 没成功，则addWaiter\(\)将该线程加入等待队列的尾部，并标记为独占模式；
3. acquireQueued\(\)使线程在等待队列中休息，有机会时（轮到自己，会被unpark\(\)）会去尝试获取资源。获取到资源后才返回。如果在整个等待过程中被中断过，则返回true，否则返回false。
4. 如果线程在等待过程中被中断过，它是不响应的。只是获取资源后才再进行自我中断selfInterrupt\(\)，将中断补上。

由于此函数是重中之重，我再用流程图总结一下：

721070-20151102145743461-623794326.png

至此，acquire\(\)的流程终于算是告一段落了。这也就是ReentrantLock.lock\(\)的流程，不信你去看其lock\(\)源码吧，整个函数就是一条acquire\(1\)！！！

## 3.2 release\(int\)

上一小节已经把acquire\(\)说完了，这一小节就来讲讲它的反操作release\(\)吧。此方法是独占模式下线程释放共享资源的顶层入口。它会释放指定量的资源，如果彻底释放了（即state=0）,它会唤醒等待队列里的其他线程来获取资源。这也正是unlock\(\)的语义，当然不仅仅只限于unlock\(\)。下面是release\(\)的源码：

```
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;//找到头结点
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);//唤醒等待队列里的下一个线程
        return true;
    }
    return false;
}
```

逻辑并不复杂。它调用tryRelease\(\)来释放资源。有一点需要注意的是，**它是根据tryRelease\(\)的返回值来判断该线程是否已经完成释放掉资源了！所以自定义同步器在设计tryRelease\(\)的时候要明确这一点！！**

### 3.2.1 tryRelease\(int\)

此方法尝试去释放指定量的资源。下面是tryRelease\(\)的源码：

```
protected boolean tryRelease(int arg) {
    throw new UnsupportedOperationException();
 }
```

跟tryAcquire\(\)一样，这个方法是需要独占模式的自定义同步器去实现的。正常来说，tryRelease\(\)都会成功的，因为这是独占模式，该线程来释放资源，那么它肯定已经拿到独占资源了，直接减掉相应量的资源即可\(state-=arg\)，也不需要考虑线程安全的问题。但要注意它的返回值，上面已经提到了，**release\(\)是根据tryRelease\(\)的返回值来判断该线程是否已经完成释放掉资源了！**所以自义定同步器在实现时，如果已经彻底释放资源\(state=0\)，要返回true，否则返回false。

### 3.2.2 unparkSuccessor\(Node\)

此方法用于唤醒等待队列中下一个线程。下面是源码：

```
private void unparkSuccessor(Node node) {
    //这里，node一般为当前线程所在的结点。
    int ws = node.waitStatus;
    if (ws < 0)//置零当前线程所在的结点状态，允许失败。
        compareAndSetWaitStatus(node, ws, 0);

    Node s = node.next;//找到下一个需要唤醒的结点s
    if (s == null || s.waitStatus > 0) {//如果为空或已取消
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)//从这里可以看出，<=0的结点，都是还有效的结点。
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);//唤醒
}
```

这个函数并不复杂。一句话概括：**用unpark\(\)唤醒等待队列中最前边的那个未放弃线程**，这里我们也用s来表示吧。此时，再和acquireQueued\(\)联系起来，s被唤醒后，进入if \(p == head && tryAcquire\(arg\)\)的判断（即使p!=head也没关系，它会再进入shouldParkAfterFailedAcquire\(\)寻找一个安全点。这里既然s已经是等待队列中最前边的那个未放弃线程了，那么通过shouldParkAfterFailedAcquire\(\)的调整，s也必然会跑到head的next结点，下一次自旋p==head就成立啦），然后s把自己设置成head标杆结点，表示自己已经获取到资源了，acquire\(\)也返回了！！And then, DO what you WANT!

### 3.2.3 小结

release\(\)是独占模式下线程释放共享资源的顶层入口。它会释放指定量的资源，如果彻底释放了（即state=0）,它会唤醒等待队列里的其他线程来获取资源。

## 3.3 acquireShared\(int\)

此方法是共享模式下线程获取共享资源的顶层入口。它会获取指定量的资源，获取成功则直接返回，获取失败则进入等待队列，直到获取到资源为止，整个过程忽略中断。下面是acquireShared\(\)的源码：

```
1 public final void acquireShared(int arg) {
2     if (tryAcquireShared(arg) < 0)
3         doAcquireShared(arg);
4 }
```







