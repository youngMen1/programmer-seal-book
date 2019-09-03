# [JAVA中写时复制\(Copy-On-Write\)Map实现](https://www.cnblogs.com/hapjin/p/4840107.html)

## 1，什么是写时复制\(Copy-On-Write\)容器？

写时复制是指：在并发访问的情景下，当需要修改JAVA中Containers的元素时，不直接修改该容器，而是先复制一份副本，在副本上进行修改。修改完成之后，将指向原来容器的引用指向新的容器\(副本容器\)。

## 2，写时复制带来的影响

①由于不会修改原始容器，只修改副本容器。因此，可以对原始容器进行并发地读。其次，实现了读操作与写操作的分离，读操作发生在原始容器上，写操作发生在副本容器上。

②数据一致性问题：读操作的线程可能不会立即读取到新修改的数据，因为修改操作发生在副本上。但最终修改操作会完成并更新容器，因此这是最终一致性。

## 3，在JDK中提供了CopyOnWriteArrayList类和CopyOnWriteArraySet类，但是并没有提供CopyOnWriteMap的实现。因此，可以参考CopyOnWriteArrayList自己实现一个CopyOnWriteHashMap

这里主要是实现 在写操作时，如何保证线程安全。

```
import java.util.Collection;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;


public class CopyOnWriteMap<K, V> implements Map<K, V>, Cloneable{

    private volatile Map<K, V> internalMap;
    
    public CopyOnWriteMap() {
        internalMap = new HashMap<K, V>(100);//初始大小应根据实际应用来指定
    }
    
    @Override
    public V put(K key, V value) {
        synchronized (this) {
            Map<K, V> newMap = new HashMap<K, V>(internalMap);//复制出一个新HashMap
            V val = newMap.put(key, value);//在新HashMap中执行写操作
            internalMap = newMap;//将原来的Map引用指向新Map
            return val;
        }
    }
    
    @Override
    public void putAll(Map<? extends K, ? extends V> m) {
        synchronized (this) {
            Map<K, V> newMap = new HashMap<K, V>(internalMap);
            newMap.putAll(m);
            internalMap = newMap;
        }
        
    }
    
    @Override
    public V get(Object key) {
        V result = internalMap.get(key);
        return result;
    }
    ......//other methods inherit from interface Map
}
```



