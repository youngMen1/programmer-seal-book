## AQS简介

在\[上一篇文章\]\([https://juejin.im/post/5aeb055b6fb9a07abf725c8c\)中我们对lock和AbstractQueuedSynchronizer\(AQS\)有了初步的认识。在同步组件的实现中，AQS是核心部分，同步组件的实现者通过使用AQS提供的模板方法实现同步组件语义，AQS则实现了对\*\*同步](https://juejin.im/post/5aeb055b6fb9a07abf725c8c%29中我们对lock和AbstractQueuedSynchronizer%28AQS%29有了初步的认识。在同步组件的实现中，AQS是核心部分，同步组件的实现者通过使用AQS提供的模板方法实现同步组件语义，AQS则实现了对**同步)状态的管理，以及对阻塞线程进行排队，等待通知\*\*等等一些底层的实现处理。AQS的核心也包括了这些方面:\*\***同步队列，独占式锁的获取和释放，共享锁的获取和释放以及可中断锁，超时等待锁获取这些特性的实现**\*\*，而这些实际上则是AQS提供出来的模板方法，归纳整理如下：

\*\***独占式锁：**\*\*

&gt; void acquire\(int arg\)：独占式获取同步状态，如果获取失败则插入同步队列进行等待；

&gt; void acquireInterruptibly\(int arg\)：与acquire方法相同，但在同步队列中进行等待的时候可以检测中断；

&gt; boolean tryAcquireNanos\(int arg, long nanosTimeout\)：在acquireInterruptibly基础上增加了超时等待功能，在超时时间内没有获得同步状态返回false;

&gt; boolean release\(int arg\)：释放同步状态，该方法会唤醒在同步队列中的下一个节点

\*\***共享式锁：**\*\*

&gt; void acquireShared\(int arg\)：共享式获取同步状态，与独占式的区别在于同一时刻有多个线程获取同步状态；

&gt; void acquireSharedInterruptibly\(int arg\)：在acquireShared方法基础上增加了能响应中断的功能；

&gt; boolean tryAcquireSharedNanos\(int arg, long nanosTimeout\)：在acquireSharedInterruptibly基础上增加了超时等待的功能；

&gt; boolean releaseShared\(int arg\)：共享式释放同步状态

要想掌握AQS的底层实现，其实也就是对这些模板方法的逻辑进行学习。在学习这些模板方法之前，我们得首先了解下AQS中的同步队列是一种什么样的数据结构，因为同步队列是AQS对同步状态的管理的基石。

## 2. 同步队列

当共享资源被某个线程占有，其他请求该资源的线程将会阻塞，从而进入同步队列。就数据结构而言，队列的实现方式无外乎两者一是通过数组的形式，另外一种则是链表的形式。AQS中的同步队列则是\*\*通过链式方式\*\*进行实现。接下来，很显然我们至少会抱有这样的

疑问：\*\*1. 节点的数据结构是什么样的？2. 是单向还是双向？3. 是带头结点的还是不带头节点的？\*\*我们依旧先是通过看源码的方式。

在AQS有一个静态内部类Node，其中有这样一些属性：

&gt; volatile int waitStatus //节点状态

&gt; volatile Node prev //当前节点/线程的前驱节点

&gt; volatile Node next; //当前节点/线程的后继节点

&gt; volatile Thread thread;//加入同步队列的线程引用

&gt; Node nextWaiter;//等待队列中的下一个节点

节点的状态有以下这些：

&gt; int CANCELLED =  1//节点从同步队列中取消

&gt; int SIGNAL    = -1//后继节点的线程处于等待状态，如果当前节点释放同步状态会通知后继节点，使得后继节点的线程能够运行；

&gt; int CONDITION = -2//当前节点进入等待队列中

&gt; int PROPAGATE = -3//表示下一次共享式同步状态获取将会无条件传播下去

&gt; int INITIAL = 0;//初始状态

现在我们知道了节点的数据结构类型，并且每个节点拥有其前驱和后继节点，很显然这是\*\***一个双向队列**\*\*。同样的我们可以用一段demo看一下。

```
    public class LockDemo {
        private static ReentrantLock lock = new ReentrantLock();

        public static void main(String[] args) {
            for (int i = 0; i < 5; i++) {
                Thread thread = new Thread(() -> {
                    lock.lock();
                    try {
                        Thread.sleep(10000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } finally {
                        lock.unlock();
                    }
                });
                thread.start();
            }
        }
    }
```

实例代码中开启了5个线程，先获取锁之后再睡眠10S中，实际上这里让线程睡眠是想模拟出当线程无法获取锁时进入同步队列的情况。通过debug，当Thread-4（在本例中最后一个线程）获取锁失败后进入同步时，AQS时现在的同步队列如图所示：

![](/assets/LockDemo debug下.png)

!\[LockDemo debug下 .png\]\([http://upload-images.jianshu.io/upload\_images/2615789-d05d3f44ce4c205a.png?imageMogr2/auto-orient/strip\|imageView2/2/w/1240\](http://upload-images.jianshu.io/upload_images/2615789-d05d3f44ce4c205a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240%29\)

Thread-0先获得锁后进行睡眠，其他线程（Thread-1,Thread-2,Thread-3,Thread-4）获取锁失败进入同步队列，同时也可以很清楚的看出来每个节点有两个域：prev\(前驱\)和next\(后继\)，并且每个节点用来保存获取同步状态失败的线程引用以及等待状态等信息。另外

AQS中有两个重要的成员变量：

```
    private transient volatile Node head;
    private transient volatile Node tail;
```

也就是说AQS实际上通过头尾指针来管理同步队列，同时实现包括获取锁失败的线程进行入队，释放锁时对同步队列中的线程进行通知等核心方法。其示意图如下：

![](/assets/队列的示意图.png)

!\[队列示意图.png\]\([http://upload-images.jianshu.io/upload\_images/2615789-dbfc975d3601bb52.png?imageMogr2/auto-orient/strip\|imageView2/2/w/1240\](http://upload-images.jianshu.io/upload_images/2615789-dbfc975d3601bb52.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240%29\)

通过对源码的理解以及做实验的方式，现在我们可以清楚的知道这样几点：

1. \*\***节点的数据结构，即AQS的静态内部类Node,节点的等待状态等信息**\*\*；

2. \*\***同步队列是一个双向队列，AQS通过持有头尾指针管理同步队列**\*\*；

那么，节点如何进行入队和出队是怎样做的了？实际上这对应着锁的获取和释放两个操作：获取锁失败进行入队操作，获取锁成功进行出队操作。

## 3. 独占锁

### 3.1 独占锁的获取（acquire方法）

我们继续通过看源码和debug的方式来看，还是以上面的demo为例，调用lock\(\)方法是获取独占式锁，获取失败就将当前线程加入同步队列，成功则线程执行。而lock\(\)方法实际上会调用AQS的\*\***acquire\(\)**\*\*方法，源码如下

```
    public final void acquire(int arg) {
            //先看同步状态是否获取成功，如果成功则方法结束返回
            //若失败则先调用addWaiter()方法再调用acquireQueued()方法
            if (!tryAcquire(arg) &&
                acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
                selfInterrupt();
    }
```

关键信息请看注释，acquire根据当前获得同步状态成功与否做了两件事情：1. 成功，则方法结束返回，2. 失败，则先调用addWaiter\(\)然后在调用acquireQueued\(\)方法。

&gt; \*\***获取同步状态失败，入队操作**\*\*

当线程获取独占式锁失败后就会将当前线程加入同步队列，那么加入队列的方式是怎样的了？我们接下来就应该去研究一下addWaiter\(\)和acquireQueued\(\)。addWaiter\(\)源码如下：

```
private Node addWaiter(Node mode) {
            // 1. 将当前线程构建成Node类型
            Node node = new Node(Thread.currentThread(), mode);
            // Try the fast path of enq; backup to full enq on failure
            // 2. 当前尾节点是否为null？
            Node pred = tail;
            if (pred != null) {
                // 2.2 将当前节点尾插入的方式插入同步队列中
                node.prev = pred;
                if (compareAndSetTail(pred, node)) {
                    pred.next = node;
                    return node;
                }
            }
            // 2.1. 当前同步队列尾节点为null，说明当前线程是第一个加入同步队列进行等待的线程
            enq(node);
            return node;
    }
```

分析可以看上面的注释。程序的逻辑主要分为两个部分：\*\*1. 当前同步队列的尾节点为null，调用方法enq\(\)插入;2. 当前队列的尾节点不为null，则采用尾插入（compareAndSetTail（）方法）的方式入队。\*\*另外还会有另外一个问题：如果 \`if \(compareAndSetTail

\(pred, node\)\)\`为false怎么办？会继续执行到enq\(\)方法，同时很明显compareAndSetTail是一个CAS操作，通常来说如果CAS操作失败会继续自旋（死循环）进行重试。因此，经过我们这样的分析，enq\(\)方法可能承担两个任务：\*\***1. 处理当前同步队列尾节点为null时进行入队操作；2. 如果CAS尾插入节点失败后负责自旋进行尝试。**\*\*那么是不是真的就像我们分析的一样了？只有源码会告诉我们答案:\),enq\(\)源码如下：

```
    private Node enq(final Node node) {
            for (;;) {
                Node t = tail;
                if (t == null) { // Must initialize
                    //1. 构造头结点
                    if (compareAndSetHead(new Node()))
                        tail = head;
                } else {
                    // 2. 尾插入，CAS操作失败自旋尝试
                    node.prev = t;
                    if (compareAndSetTail(t, node)) {
                        t.next = node;
                        return t;
                    }
                }
            }
    }
```

在上面的分析中我们可以看出在第1步中会先创建头结点，说明同步队列是\*\***带头结点的链式存储结构**\*\*。带头结点与不带头结点相比，会在入队和出队的操作中获得更大的便捷性，因此同步队列选择了带头结点的链式存储结构。那么带头节点的队列初始化时机是什么？自然而然是在\*\*tail为null时，即当前线程是第一次插入同步队列\*\*。compareAndSetTail\(t, node\)方法会利用CAS操作设置尾节点，如果CAS操作失败会在\`for \(;;\)\`for死循环中不断尝试，直至成功return返回为止。因此，对enq\(\)方法可以做这样的总结：

1. \*\***在当前线程是第一个加入同步队列时，调用compareAndSetHead\(new Node\(\)\)方法，完成链式队列的头结点的初始化**\*\*；

2. \*\***自旋不断尝试CAS尾插入节点直至成功为止**\*\*。

现在我们已经很清楚获取独占式锁失败的线程包装成Node然后插入同步队列的过程了？那么紧接着会有下一个问题？在同步队列中的节点（线程）会做什么事情了来保证自己能够有机会获得独占式锁了？带着这样的问题我们就来看看acquireQueued\(\)方法，从方法名就可以很清楚，这个方法的作用就是排队获取锁的过程，源码如下：

```
    final boolean acquireQueued(final Node node, int arg) {
            boolean failed = true;
            try {
                boolean interrupted = false;
                for (;;) {
                    // 1. 获得当前节点的先驱节点
                    final Node p = node.predecessor();
                    // 2. 当前节点能否获取独占式锁                    
                    // 2.1 如果当前节点的先驱节点是头结点并且成功获取同步状态，即可以获得独占式锁
                    if (p == head && tryAcquire(arg)) {
                        //队列头指针用指向当前节点
                        setHead(node);
                        //释放前驱节点
                        p.next = null; // help GC
                        failed = false;
                        return interrupted;
                    }
                    // 2.2 获取锁失败，线程进入等待状态等待获取独占式锁
                    if (shouldParkAfterFailedAcquire(p, node) &&
                        parkAndCheckInterrupt())
                        interrupted = true;
                }
            } finally {
                if (failed)
                    cancelAcquire(node);
            }
    }
```

程序逻辑通过注释已经标出，整体来看这是一个这又是一个自旋的过程（for \(;;\)），代码首先获取当前节点的先驱节点，\*\***如果先驱节点是头结点的并且成功获得同步状态的时候（if \(p == head && tryAcquire\(arg\)\)），当前节点所指向的线程能够获取锁**\*\*。反之，获取锁失败进入等待状态。整体示意图为下图：

![](/assets/自旋的获取锁整体示意图.png)

!\[自旋获取锁整体示意图.png\]\([http://upload-images.jianshu.io/upload\_images/2615789-3fe83cfaf03a02c8.png?imageMogr2/auto-orient/strip\|imageView2/2/w/1240\](http://upload-images.jianshu.io/upload_images/2615789-3fe83cfaf03a02c8.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240%29\)

&gt; \*\***获取锁成功，出队操作**\*\*

获取锁的节点出队的逻辑是：

