# 1.HashMap的底层实现和原理

HashMap基于Map接口实现，元素以键值对的方式存储，并且允许使用null 建和null值，因为key不允许重复，因此只能有一个键为null,另外HashMap不能保证放入元素的顺序，它是无序的，和放入的顺序并不能相同。HashMap是线程不安全的，如果需要考虑并发，则需要使用ConcurrentHashMap。

HashMap结构

JDK1.7：Entry数组+链表

JDK1.8：Entry数组+链表红黑树

**继承关系**

```
public class HashMap<K,V>extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable
```

基本属性

```
// 哈希数组最大容量
static final int MAXIMUM_CAPACITY = 1 << 30;
// 哈希数组默认容量
static final int DEFAULT_INITIAL_CAPACITY = 16;
// HashMap默认负载因子（负荷系数）     
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```



