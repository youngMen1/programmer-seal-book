## 1. synchronized简介

在学习知识前，我们先来看一个现象：

```
public class SynchronizedDemo implements Runnable {
        private static int count = 0;

        public static void main(String[] args) {
            for (int i = 0; i < 10; i++) {
                Thread thread = new Thread(new SynchronizedDemo());
                thread.start();
            }
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("result: " + count);
        }

        @Override
        public void run() {
            for (int i = 0; i < 1000000; i++)
                count++;
        }
    }
```

开启了10个线程，每个线程都累加了1000000次，如果结果正确的话自然而然总数就应该是10 \* 1000000 = 10000000。可就运行多次结果都不是这个数，而且每次运行结果都不一样。这是为什么了？有什么解决方案了？这就是我们今天要聊的事情。

在上一篇博文中我们已经了解了\[java内存模型\]\([https://juejin.im/post/5ae6d309518825673123fd0e\)的一些知识，并且已经知道出现线程安全的主要来源于JMM的设计，主要集中在主内存和线程的工作内存而导致的\*\*内存可见性问题\*\*，以及\*\*重排序导致的问题\*\*，进一步知道了\*\*happens-before规则\*\*。线程运行时拥有自己的栈空间，会在自己的栈空间运行，如果多线程间没有共享的数据也就是说多线程间并没有协作完成一件事情，那么，多线程就不能发挥优势，不能带来巨大的价值。那么共享数据的线程安全问题怎样处理？很自然而然的想法就是每一个线程依次去读写这个共享变量，这样就不会有任何数据安全的问题，因为每个线程所操作的都是当前最新的版本数据。那么，在java关键字synchronized就具有使每个线程依次排队操作共享变量的功能。很显然，这种同步机制效率很低，但synchronized是其他并发容器实现的基础，对它的理解也会大大提升对并发编程的感觉，从功利的角度来说，这也是面试高频的考点。好了，下面，就来具体说说这个关键字。](https://juejin.im/post/5ae6d309518825673123fd0e%29的一些知识，并且已经知道出现线程安全的主要来源于JMM的设计，主要集中在主内存和线程的工作内存而导致的**内存可见性问题**，以及**重排序导致的问题**，进一步知道了**happens-before规则**。线程运行时拥有自己的栈空间，会在自己的栈空间运行，如果多线程间没有共享的数据也就是说多线程间并没有协作完成一件事情，那么，多线程就不能发挥优势，不能带来巨大的价值。那么共享数据的线程安全问题怎样处理？很自然而然的想法就是每一个线程依次去读写这个共享变量，这样就不会有任何数据安全的问题，因为每个线程所操作的都是当前最新的版本数据。那么，在java关键字synchronized就具有使每个线程依次排队操作共享变量的功能。很显然，这种同步机制效率很低，但synchronized是其他并发容器实现的基础，对它的理解也会大大提升对并发编程的感觉，从功利的角度来说，这也是面试高频的考点。好了，下面，就来具体说说这个关键字。)

## 2. synchronized实现原理

在java代码中使用synchronized可是使用在代码块和方法中，根据Synchronized用的位置可以有这些使用场景：

## ![](/assets/synchronized的使用场景.png)

如图，synchronized可以用在\***\*方法**\*\*上也可以使用在\***\*代码块**\*\*中，其中方法是实例方法和静态方法分别锁的是该类的实例对象和该类的对象。而使用在代码块中也可以分为三种，具体的可以看上面的表格。这里的需要注意的是：\*\***如果锁的是类对象的话，尽管new多个实例对象，但他们仍然是属于同一个类依然会被锁住，即线程之间保证同步关系**\*\*。

现在我们已经知道了怎样synchronized了，看起来很简单，拥有了这个关键字就真的可以在并发编程中得心应手了吗？爱学的你，就真的不想知道synchronized底层是怎样实现了吗？

## 2.1 对象锁（monitor）机制

现在我们来看看synchronized的具体底层实现。先写一个简单的demo:

```
public class SynchronizedDemo {
        public static void main(String[] args) {
            synchronized (SynchronizedDemo.class) {
            }
            method();
        }

        private static void method() {
        }
    }
```

上面的代码中有一个同步代码块，锁住的是类对象，并且还有一个同步静态方法，锁住的依然是该类的类对象。编译之后，切换到SynchronizedDemo.class的同级目录之后，然后用\*\*javap -v SynchronizedDemo.class\*\*查看字节码文件：

![](/assets/synchronizedDemo.class.png)

如图，上面用黄色高亮的部分就是需要注意的部分了，这也是添Synchronized关键字之后独有的。执行同步代码块后首先要先执行\*\***monitorenter**\*\*指令，退出的时候\*\***monitorexit**\*\*指令。通过分析之后可以看出，使用Synchronized进行同步，其关键就是必须要对对象的监视器monitor进行获取，当线程获取monitor后才能继续往下执行，否则就只能等待。而这个获取的过程是\*\***互斥**\*\*的，即同一时刻只有一个线程能够获取到monitor。上面的demo中在执行完同步代码块之后紧接着再会去执行一个静态同步方法，而这个方法锁的对象依然就这个类对象，那么这个正在执行的线程还需要获取该锁吗？答案是不必的，从上图中就可以看出来，执行静态同步方法的时候就只有一条monitorexit指令，并没有monitorenter获取锁的指令。这就是\*\***锁的重入性**\*\*，即在同一锁程中，线程不需要再次获取同一把锁。Synchronized先天具有重入性。\*\***每个对象拥有一个计数器，当线程获取该对象锁后，计数器就会加一，释放锁后就会将计数器减一**\*\*。

任意一个对象都拥有自己的监视器，当这个对象由同步块或者这个对象的同步方法调用时，执行方法的线程必须先获取该对象的监视器才能进入同步块和同步方法，如果没有获取到监视器的线程将会被阻塞在同步块和同步方法的入口处，进入到BLOCKED状态（关于线程的状态可以看\[这篇文章\]\([https://juejin.im/post/5ae6cf7a518825670960fcc2\](https://juejin.im/post/5ae6cf7a518825670960fcc2%29\)

下图表现了对象，对象监视器，同步队列以及执行线程状态之间的关系：

![](/assets/对象，对象监视器，同步队列和线程状态的关系.png)

该图可以看出，任意线程对Object的访问，首先要获得Object的监视器，如果获取失败，该线程就进入同步状态，线程状态变为BLOCKED，当Object的监视器占有者释放后，在同步队列中得线程就会有机会重新获取该监视器。

## 2.2 synchronized的happens-before关系

在上一篇文章中讨论过\[happens-before\]\([https://juejin.im/post/5ae6d309518825673123fd0e\)规则，抱着学以致用的原则我们现在来看一看Synchronized的happens-before规则，即监视器锁规则：对同一个监视器的解锁，happens-before于对该监视器的加锁。继续来看代码：](https://juejin.im/post/5ae6d309518825673123fd0e%29规则，抱着学以致用的原则我们现在来看一看Synchronized的happens-before规则，即监视器锁规则：对同一个监视器的解锁，happens-before于对该监视器的加锁。继续来看代码：)

```
public class MonitorDemo {
        private int a = 0;

        public synchronized void writer() {     // 1
            a++;                                // 2
        }                                       // 3

        public synchronized void reader() {    // 4
            int i = a;                         // 5
        }                                      // 6
    }
```

该代码的happens-before关系如图所示：

在图中每一个箭头连接的两个节点就代表之间的happens-before关系，黑色的是通过程序顺序规则推导出来，红色的为监视器锁规则推导而出：\*\*线程A释放锁happens-before线程B加锁\*\*，蓝色的则是通过程序顺序规则和监视器锁规则推测出来happens-befor关系，通过传递性规则进一步推导的happens-before关系。现在我们来重点关注2 happens-before 5，通过这个关系我们可以得出什么？

根据happens-before的定义中的一条:如果A happens-before B，则A的执行结果对B可见，并且A的执行顺序先于B。线程A先对共享变量A进行加一，由2 happens-before 5关系可知线程A的执行结果对线程B可见即线程B所读取到的a的值为1。

## 2.3 锁获取和锁释放的内存语义

在上一篇文章提到过JMM核心为两个部分：happens-before规则以及内存抽象模型。我们分析完Synchronized的happens-before关系后，还是不太完整的，我们接下来看看基于java内存抽象模型的Synchronized的内存语义。

废话不多说依旧先上图。![](/assets/线程A写共享变量.png)

从上图可以看出，线程A会首先先从主内存中读取共享变量a=0的值然后将该变量拷贝到自己的本地内存，进行加一操作后，再将该值刷新到主内存，整个过程即为线程A 加锁--&gt;执行临界区代码--&gt;释放锁相对应的内存语义。

![](/assets/线程B读共享变量.png)









