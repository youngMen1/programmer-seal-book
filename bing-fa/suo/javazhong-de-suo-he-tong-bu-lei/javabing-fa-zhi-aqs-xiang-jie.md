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

