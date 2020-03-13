## 1、列举一个常用的Redis客户端的并发模型。 {#1、列举一个常用的Redis客户端的并发模型。}

```
lock = 0;
while (timeout > 0) {
  if (setnxexpire(key, value)) {
    lock = 1;
    return lock;
  }
  timeout -= sleeptime
  sleep(sleeptime);
}
```

## 2、HBase如何实现模糊查询？ {#2、HBase如何实现模糊查询？}

```
try {  
    HTable table = new HTable(conf, tablename);  
    Scan s = new Scan();
    // 查询rowkey包括xx的行
    Filter filter = new RowFilter(CompareFilter.CompareOp.EQUAL, new SubstringComparator("xx"));
    s.setFilter(filter);  
    ResultScanner rs = table.getScanner(s);  
    for (Result r : rs) {  
       KeyValue[] kv = r.raw();  
       for (int i = 0; i < kv.length; i++) {  
           System.out.print(new String(kv[i].getRow()) + "  ");  
           System.out.print(new String(kv[i].getFamily()) + ":");  
           System.out.print(new String(kv[i].getQualifier()) + "  ");  
           System.out.print(kv[i].getTimestamp() + "  ");  
           System.out.println(new String(kv[i].getValue()));  
       }  
   }  
} catch (IOException e) {
}
```

## 3、列举一个常用的消息中间件，如果消息要保序如何实现？ {#3、列举一个常用的消息中间件，如果消息要保序如何实现？}

ActiveMQ、RabbitMQ、kafka

实现队列，先进先出

## 4、如何实现一个Hashtable？你的设计如何考虑Hash冲突？如何优化？ {#4、如何实现一个Hashtable？你的设计如何考虑Hash冲突？如何优化？}

使用哈希表

[Hash算法解决冲突的方法](http://blog.csdn.net/feinik/article/details/54974293)

[处理散列冲突：开放定址法](http://lib.csdn.net/article/datastructure/30401)

开放定址法、再哈希法、链地址法、建立公共溢出区

参考Hashmap的链地址法，链表长度大于8时转为红黑树

## 5、分布式缓存，一致性hash {#5、分布式缓存，一致性hash}

[分布式缓存的一致性Hash算法](http://blog.csdn.net/onpwerb/article/details/52705307)

（1）先构造一个长度为0~2^32的整数环，根据节点名称的Hash值，将缓存服务器节点放置在这个Hash环上。

（2）根据需要缓存的数据的KEY值计算得到其Hash值，然后在Hash环上顺时针查找距离这个KEY值的Hash值最近的缓存服务器节点，完成KEY到服务器的Hash映射查找。

补充：

这个一致性Hash环使用二叉查找树实现，Hash查找过程实际上是在二叉查找树中查找不小于查找树的最小数值。

另外，为了解决上述算法带来的负载不均衡问题，通过使用虚拟层，将每台物理缓存服务器虚拟为一组虚拟缓存服务器，将虚拟服务器的Hash值放置在Hash环上，KEY在环上先找到虚拟服务器节点，再得到物理服务器的信息。

## 6、LRU算法，slab分配，如何减少内存碎片 {#6、LRU算法，slab分配，如何减少内存碎片}

[两种常见的缓存淘汰算法LFU&LRU](http://blog.csdn.net/jake_li/article/details/50659868)

[内存碎片和memcached slab控制碎片方法](http://blog.csdn.net/u010412301/article/details/52471424)

## 7、如何解决缓存单机热点问题 {#7、如何解决缓存单机热点问题}

[如何解决高并发下缓存被击穿的问题](http://blog.csdn.net/cainiao_user/article/details/78301563)

## 8、什么是布隆过滤器，其实现原理是？ False positive指的是？ {#8、什么是布隆过滤器，其实现原理是？-False-positive指的是？}

[Bloom Filter概念和原理](http://blog.csdn.net/jiaomeng/article/details/1495500)

Bloom Filter是一种空间效率很高的随机数据结构，它利用位数组很简洁地表示一个集合，并能判断一个元素是否属于这个集合。Bloom Filter的这种高效是有一定代价的：在判断一个元素是否属于某个集合时，有可能会把不属于这个集合的元素误认为属于这个集合（false positive）。因此，Bloom Filter不适合那些“零错误”的应用场合。而在能容忍低错误率的应用场合下，Bloom Filter通过极少的错误换取了存储空间的极大节省。

## 9、memcache与redis的区别 {#9、memcache与redis的区别}

[Redis和Memcache的区别分析](http://blog.csdn.net/u013474436/article/details/48632665)

## 10、zookeeper有什么功能，选举算法如何进行 {#10、zookeeper有什么功能，选举算法如何进行}

zookeeper是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务等。

[【分布式】Zookeeper的Leader选举](http://www.cnblogs.com/leesf456/p/6107600.html)

## 11、map/reduce过程，如何用map/reduce实现两个数据源的联合统计 {#11、map-reduce过程，如何用map-reduce实现两个数据源的联合统计}

[Hadoop Map/Reduce教程](http://hadoop.apache.org/docs/r1.0.4/cn/mapred_tutorial.html)

