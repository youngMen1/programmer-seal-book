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
// 默认初始容量16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; 
// 容量最大值
static final int MAXIMUM_CAPACITY = 1 << 30;
// 默认加载因子0.75
static final float DEFAULT_LOAD_FACTOR = 0.75f;
// 树化的阈值，当桶中链表节点数大于8时，将链表转换为红黑树
static final int TREEIFY_THRESHOLD = 8;
// 红黑树退化为链表的阈值，当桶中红黑树节点数小于6时，将红黑树转换为链表
static final int UNTREEIFY_THRESHOLD = 6;
// 最小的树化容量，进行树化的时候，还有一次判断，只有键值对数量大于64时才会发生转换，
// 这是为了避免在哈希表建立初期，多个键值对恰好被放入了同一个链表而导致不必要的转化
static final int MIN_TREEIFY_CAPACITY = 64;
```

![](/static/image/20190615101545943.png)

