# 1.JDK1.8HashMap的底层实现和原理

HashMap基于Map接口实现，元素以键值对的方式存储，并且允许使用null 建和null值，因为key不允许重复，因此只能有一个键为null,另外HashMap不能保证放入元素的顺序，它是无序的，和放入的顺序并不能相同。HashMap是线程不安全的，如果需要考虑并发，则需要使用ConcurrentHashMap。

![](/static/image/20190615101545943.png)

HashMap结构

JDK1.7：Entry数组 + 链表

JDK1.8：Entry数组 + 链表 + 红黑树

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

put\(\)方法简单解析：

resize\(\)方法简单解析：

```
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



