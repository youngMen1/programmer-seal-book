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

 

在上一篇博文中我们已经了解了\[java内存模型\]\(https://juejin.im/post/5ae6d309518825673123fd0e\)的一些知识，并且已经知道出现线程安全的主要来源于JMM的设计，主要集中在主内存和线程的工作内存而导致的\*\*内存可见性问题\*\*，以及\*\*重排序导致的问题\*\*，进一步知道了\*\*happens-before规则\*\*。线程运行时拥有自己的栈空间，会在自己的栈空间运行，如果多线程间没有共享的数据也就是说多线程间并没有协作完成一件事情，那么，多线程就不能发挥优势，不能带来巨大的价值。那么共享数据的线程安全问题怎样处理？很自然而然的想法就是每一个线程依次去读写这个共享变量，这样就不会有任何数据安全的问题，因为每个线程所操作的都是当前最新的版本数据。那么，在java关键字synchronized就具有使每个线程依次排队操作共享变量的功能。很显然，这种同步机制效率很低，但synchronized是其他并发容器实现的基础，对它的理解也会大大提升对并发编程的感觉，从功利的角度来说，这也是面试高频的考点。好了，下面，就来具体说说这个关键字。

\# 2. synchronized实现原理 \#

在java代码中使用synchronized可是使用在代码块和方法中，根据Synchronized用的位置可以有这些使用场景：



