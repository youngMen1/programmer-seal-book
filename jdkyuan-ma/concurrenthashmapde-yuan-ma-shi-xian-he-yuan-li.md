# 1.ConcurrentHashMap的源码实现和原理

ConcurrentHashMap使用了锁分段（减小锁范围）、CAS（乐观锁，减小上下文切换开销，无阻塞）等等技术

## 1.1.ConcurrentHashMap源码如何实现的

### 1.1.1.ConcurrentHashMap在jdk1.7中的设计

先简单看下ConcurrentHashMap类在jdk1.7中的设计，其基本结构如图所示：
![](/static/image/764863-20160620202714522-1795796503.png)
每一个segment都是一个`HashEntry<K,V>[] table`， table中的每一个元素本质上都是一个HashEntry的单向队列。比如table[3]为首节点，table[3]->next为节点1，之后为节点2，依次类推。



### 1.1.2.ConcurrentHashMap在jdk1.8中的设计








## 1.2.如何实现线程安全的

## 1.3.jdk1.7和jdk1.8的区别

## 1.4.ConcurrentHashMap使用问题

# 2.总结
在ConcurrentHashMap没有出现以前，jdk使用hashtable来实现线程安全，但是hashtable是将整个hash表锁住，所以效率很低下。

# 3.参考


