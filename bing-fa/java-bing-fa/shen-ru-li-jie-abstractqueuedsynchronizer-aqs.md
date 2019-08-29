## AQS简介

在\[上一篇文章\]\(https://juejin.im/post/5aeb055b6fb9a07abf725c8c\)中我们对lock和AbstractQueuedSynchronizer\(AQS\)有了初步的认识。在同步组件的实现中，AQS是核心部分，同步组件的实现者通过使用AQS提供的模板方法实现同步组件语义，AQS则实现了对\*\*同步

状态的管理，以及对阻塞线程进行排队，等待通知\*\*等等一些底层的实现处理。AQS的核心也包括了这些方面:\*\*同步队列，独占式锁的获取和释放，共享锁的获取和释放以及可中断锁，超时等待锁获取这些特性的实现\*\*，而这些实际上则是AQS提供出来的模板方法，归纳

整理如下：



\*\*独占式锁：\*\*



&gt; void acquire\(int arg\)：独占式获取同步状态，如果获取失败则插入同步队列进行等待；

&gt; void acquireInterruptibly\(int arg\)：与acquire方法相同，但在同步队列中进行等待的时候可以检测中断；

&gt; boolean tryAcquireNanos\(int arg, long nanosTimeout\)：在acquireInterruptibly基础上增加了超时等待功能，在超时时间内没有获得同步状态返回false;

&gt; boolean release\(int arg\)：释放同步状态，该方法会唤醒在同步队列中的下一个节点



\*\*共享式锁：\*\*

&gt; void acquireShared\(int arg\)：共享式获取同步状态，与独占式的区别在于同一时刻有多个线程获取同步状态；

&gt; void acquireSharedInterruptibly\(int arg\)：在acquireShared方法基础上增加了能响应中断的功能；

&gt; boolean tryAcquireSharedNanos\(int arg, long nanosTimeout\)：在acquireSharedInterruptibly基础上增加了超时等待的功能；

&gt; boolean releaseShared\(int arg\)：共享式释放同步状态





要想掌握AQS的底层实现，其实也就是对这些模板方法的逻辑进行学习。在学习这些模板方法之前，我们得首先了解下AQS中的同步队列是一种什么样的数据结构，因为同步队列是AQS对同步状态的管理的基石。



\# 2. 同步队列 \#

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



现在我们知道了节点的数据结构类型，并且每个节点拥有其前驱和后继节点，很显然这是\*\*一个双向队列\*\*。同样的我们可以用一段demo看一下。



