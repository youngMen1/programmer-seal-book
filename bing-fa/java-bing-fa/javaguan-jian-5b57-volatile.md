## 1. volatile简介

在上一篇文章中我们深入理解了java关键字\[synchronized\]\([https://juejin.im/post/5ae6dc04f265da0ba351d3ff\)，我们知道在java中还有一大神器就是关键volatile，可以说是和synchronized各领风骚，其中奥妙，我们来共同探讨下。](https://juejin.im/post/5ae6dc04f265da0ba351d3ff%29，我们知道在java中还有一大神器就是关键volatile，可以说是和synchronized各领风骚，其中奥妙，我们来共同探讨下。)

通过上一篇的文章我们了解到synchronized是阻塞式同步，在线程竞争激烈的情况下会升级为重量级锁。而volatile就可以说是java虚拟机提供的最轻量级的同步机制。但它同时不容易被正确理解，也至于在并发编程中很多程序员遇到线程安全的问题就会使用

synchronized。\[Java内存模型\]\([https://juejin.im/post/5ae6d309518825673123fd0e\)告诉我们，各个线程会将共享变量从主内存中拷贝到工作内存，然后执行引擎会基于工作内存中的数据进行操作处理。线程在工作内存进行操作后何时会写到主内存中？这个时机对普](https://juejin.im/post/5ae6d309518825673123fd0e%29告诉我们，各个线程会将共享变量从主内存中拷贝到工作内存，然后执行引擎会基于工作内存中的数据进行操作处理。线程在工作内存进行操作后何时会写到主内存中？这个时机对普)

通变量是没有规定的，而针对volatile修饰的变量给java虚拟机特殊的约定，线程对volatile变量的修改会立刻被其他线程所感知，即不会出现数据脏读的现象，从而保证数据的“可见性”。

现在我们有了一个大概的印象就是：\*\***被volatile修饰的变量能够保证每个线程能够获取该变量的最新值，从而避免出现数据脏读的现象。**\*\*

## 2. volatile实现原理

volatile是怎样实现了？比如一个很简单的Java代码：

```
instance = new Instancce()  // instance是volatile变量
```



