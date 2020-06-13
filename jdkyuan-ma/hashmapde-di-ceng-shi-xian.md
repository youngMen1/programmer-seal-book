# 1.HashMap的底层实现和原理

HashMap基于Map接口实现，元素以键值对的方式存储，并且允许使用null 建和null　值，因为key不允许重复，因此只能有一个键为null,另外HashMap不能保证放入元素的顺序，它是无序的，和放入的顺序并不能相同。HashMap是线程不安全的。

继承关系

```
public class HashMap<K,V>extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable
 
```



