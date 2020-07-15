# 1.JDK1.8HashMap的源码实现和原理

HashMap基于Map接口实现，元素以键值对的方式存储，并且允许使用null 建和null值，因为key不允许重复，因此只能有一个键为null,另外HashMap不能保证放入元素的顺序，它是无序的，和放入的顺序并不能相同。HashMap是线程不安全的，如果需要考虑并发，则需要使用ConcurrentHashMap。

![](/static/image/20190615101545943.png)

**HashMap结构**

**JDK1.7：数组 + 链表**

1.数据采用头插法，新插入的元素都是放在了链表的头部位置，但是这种操作在高并发的环境下容易导致死锁。

2.JDK1.7中rehash的时候，旧链表迁移新链表的时候，如果在新表的数组索引位置相同，则链表元素会倒置

**JDK1.8：数组 + 链表 + 红黑树**

1.采用尾插法，数据新插入的元素都放在了链表的尾部。

2.JDK1.8不会倒置

3.提升了性能，解决了发生哈希碰撞后链表过长而导致索引效率慢的问题

4.扩容优化在扩充HashMap的时候，不需要像JDK1.7的实现那样重新计算hash，只需要看看原来的hash值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成“原索引+oldCap”

**继承关系**

```
public class HashMap<K,V>extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable
```

**基本属性**

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

# 2.方法解析

## 2.1.put\(\)方法简单解析：

```
public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
}
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        //如果表为空或者表的容量为0，resize初始化表
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        //根据hash得到在表中索引位置的桶，如果桶为空，则将节点直接插入桶中
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        //桶不为空
        else {
            Node<K,V> e; K k;
            //首先判断桶中第一个节点的hash与待插入元素的key的hash值是否相同且key是否"相等"，如果相等，赋给变量e
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            //是树节点，则调用putTreeVal添加到红黑树中
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            //否则是链表，遍历链表，如果不存在相同的key，则插入链表尾部，并且判断节点数量是否大于树化阈值，如果大于则转换为红黑树；如果存在相同的key，break，遍历链表结束
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            // 链表转化为红黑树
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            //e不为空表示存在相同的key，替换value并返回旧值
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        // 链表元素增加，并判断是否大于阈值，如果大于，则扩容
        if (++size > threshold)
           // 扩容
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

## 2.2.treeifyBin\(\)方法简单解析：

```
    /*
     * 当桶中链表节点数大于8时，将链表转换为红黑树
     *
     * tab：待转换的链表
     * hash：某元素的哈希值
     */
    final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        //如果表为空或者表的长度小于树化的容量，resize()扩容而不是树化
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            //hd是转换为树节点后桶中的头节点   tl记录上一个遍历的节点
            TreeNode<K,V> hd = null, tl = null;
            do {
                //将hash位置处的桶中的每个节点包装成树节点，p记录当前遍历的节点
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
                //循环将桶中每个节点替换为树节点，最终结果就是链表转换为双向链表，prev指向前一个节点，next指向后一个节点
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)  
                    //将双向链表转化为红黑树
                    hd.treeify(tab);
            }
    }
```

## 2.3.resize\(\)方法简单解析：

如果存在key节点，返回旧值，如果不存在则返回Null。

```
    /*
     * 初始化哈希数组，或者对哈希数组扩容，返回新的哈希数组
     *
     * 注：哈希数组的容量跟HashMap存放的元素数量没有必然联系
     *    哈希数组只存放一系列同位元素（在HashMap中占据相同位置的元素）中最早进来的那个
     */
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        //旧表的容量
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        //旧的阈值
        int oldThr = threshold;
        //记录新表的容量大小和阈值
        int newCap, newThr = 0;
        //旧表容量大于0，表示被初始化过，需要执行的是扩容操作
        if (oldCap > 0) {
            //如果旧表容量大于容量最大值，那么阈值为Interger的最大值，即提升阈值，不再进行扩容，返回旧表
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            //否则，扩容为原先容量的1倍，阈值也扩容为原来的一倍
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        //oldCap不大于0，表示该表未被初始化，需要进行初始化，需要确认表的大小及阈值
        //旧表容量为0，阈值大于0，则用阈值大小作为容量
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        //否则，表的容量为默认初始容量16，阈值为默认初始容量16*加载因子0.75
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        //如果新表阈值为0，则利用新容量*加载因子计算
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        //将新的阈值赋给HashMap的阈值成员变量
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        //新建数组，大小为newCap
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        //将新建的表赋给HashMap的表成员变量
        table = newTab;
        //如果旧表不为空，则需要进行扩容
        if (oldTab != null) {
            //变量旧表中的每一个桶
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                //将不为空的桶重hash到新表中
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    //桶中只有一个元素，将该元素放到新表的桶中
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    //桶中存放的是红黑树，复杂这里不做讲解
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    //桶中存放的是链表
                    //并没有进行
                    else { // preserve order
                        //根据变化的最高位的不同，也就是0或者1，将链表拆分开
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            //最高位为0，则将节点加入loTail.next
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            //最高位为1，则将节点加入hiTail.next
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        //在新数组的位置与原数组的位置相同,新数组的桶直接指向LoHead
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        //在新数组的位置是原数组的位置+旧数组长度,新数组的桶直接指向hiHead
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        //返回扩容后的新数组或者初始化后的数组
        return newTab;
    }
```

# 3.总结

泊松分布.jpg

# 4.参考

jdk1.8中HashMap在扩容的时候做了哪些优化

[https://cloud.tencent.com/developer/article/1571903](https://cloud.tencent.com/developer/article/1571903)

HashMap1.8较1.7做了哪些优化

[https://www.cnblogs.com/name-lizonglin/p/12090408.html](https://www.cnblogs.com/name-lizonglin/p/12090408.html)

