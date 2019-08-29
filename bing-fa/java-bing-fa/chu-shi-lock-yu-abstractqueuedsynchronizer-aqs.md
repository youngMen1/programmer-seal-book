## 1. concurrent包的结构层次

在针对并发编程中，Doug Lea大师为我们提供了大量实用，高性能的工具类，针对这些代码进行研究会让我们队并发编程的掌握更加透彻也会大大提升我们队并发编程技术的热爱。这些代码在java.util.concurrent包下。如下图，即为concurrent包的目录结构图。

![](/assets/concurrent目录结构.png)

!\[concurrent目录结构.png\]\([http://upload-images.jianshu.io/upload\_images/2615789-da951eb99c5dabfd.png?imageMogr2/auto-orient/strip\|imageView2/2/w/1240\](http://upload-images.jianshu.io/upload_images/2615789-da951eb99c5dabfd.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240%29\)

其中包含了两个子包：**atomic以及lock**，另外在concurrent下的阻塞队列以及executors,这些就是concurrent包中的精华，之后会一一进行学习。而这些类的实现主要是依赖于volatile以及CAS（关于volatile可以看\[这篇文章\]

\([https://juejin.im/post/5ae9b41b518825670b33e6c4\)，关于CAS可以看\[这篇文章的3.1节\]\(https://juejin.im/post/5ae6dc04f265da0ba351d3ff\)），从整体上来看concurrent包的整体实现图如下图所示：](https://juejin.im/post/5ae9b41b518825670b33e6c4%29，关于CAS可以看[这篇文章的3.1节]%28https://juejin.im/post/5ae6dc04f265da0ba351d3ff%29），从整体上来看concurrent包的整体实现图如下图所示：)

![](/assets/concurrent包实现整体示意图.png)





!\[concurrent包实现整体示意图.png\]\([http://upload-images.jianshu.io/upload\_images/2615789-24da822ddc226b03.png?imageMogr2/auto-orient/strip\|imageView2/2/w/1240\](http://upload-images.jianshu.io/upload_images/2615789-24da822ddc226b03.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240%29\)

## 2. lock简介

我们下来看concurent包下的lock子包。锁是用来控制多个线程访问共享资源的方式，一般来说，一个锁能够防止多个线程同时访问共享资源。在Lock接口出现之前，java程序主要是靠synchronized关键字实现锁功能的，而java SE5之后，并发包中增加了lock接口，它提

供了与synchronized一样的锁功能。\*\*虽然它失去了像synchronize关键字隐式加锁解锁的便捷性，但是却拥有了锁获取和释放的可操作性，可中断的获取锁以及超时获取锁等多种synchronized关键字所不具备的同步特性。\*\*通常使用显示使用lock的形式如下：

