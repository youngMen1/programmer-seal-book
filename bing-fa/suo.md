# [线程安全的无锁RingBuffer的实现【一个读线程，一个写线程】](https://www.cnblogs.com/l00l/p/4115001.html)

在程序设计中，我们有时会遇到这样的情况，一个线程将数据写到一个buffer中，另外一个线程从中读数据。所以这里就有多线程竞争的问题。通常的解决办法是对竞争资源加锁。但是，一般加锁的损耗较高。其实，对于这样的一个线程写，一个线程读的特殊情况，可以以一种简单的无锁RingBuffer来实现。这样代码的运行效率很高。

本文借鉴了[Disruptor](http://code.google.com/p/disruptor/%20)项目代码。

代码我在github上放了一份，需要的同学可以去下载（[RingBuffer.java](https://github.com/drneverend/buffers/blob/master/ringbuffer/RingBuffer.java)）。本文最后也会附上一份。

代码的基本原理如下。



