# 1.ConcurrentHashMap的源码实现和原理

ConcurrentHashMap使用了锁分段（减小锁范围）、CAS（乐观锁，减小上下文切换开销，无阻塞）等等技术

## 1.1.ConcurrentHashMap源码如何实现的

### 1.1.1.ConcurrentHashMap在jdk1.7中的设计

先简单看下ConcurrentHashMap类在jdk1.7中的设计，其基本结构如图所示：

![](/static/image/764863-20160620202714522-1795796503.png)

每一个segment都是一个`HashEntry<K,V>[] table`， table中的每一个元素本质上都是一个HashEntry的单向队列。比如table\[3\]为首节点，table\[3\]-&gt;next为节点1，之后为节点2，依次类推。

```
public class ConcurrentHashMap<K, V> extends AbstractMap<K, V>
        implements ConcurrentMap<K, V>, Serializable {

    // 将整个hashmap分成几个小的map，每个segment都是一个锁；与hashtable相比，这么设计的目的是对于put, remove等操作，可以减少并发冲突，对
    // 不属于同一个片段的节点可以并发操作，大大提高了性能
    final Segment<K,V>[] segments;

    // 本质上Segment类就是一个小的hashmap，里面table数组存储了各个节点的数据，继承了ReentrantLock, 可以作为互拆锁使用
    static final class Segment<K,V> extends ReentrantLock implements Serializable {
        transient volatile HashEntry<K,V>[] table;
        transient int count;
    }

    // 基本节点，存储Key， Value值
    static final class HashEntry<K,V> {
        final int hash;
        final K key;
        volatile V value;
        volatile HashEntry<K,V> next;
    }
}
```

```
 /*
     * 向当前Map中存入新的元素，并返回旧元素
     *
     * onlyIfAbsent 是否需要维持原状（不覆盖旧值）
     */
    final V putVal(K key, V value, boolean onlyIfAbsent) {
        if(key == null || value == null) {
            throw new NullPointerException();
        }

        /*
         * 计算key的哈希值，在这个过程中会调用key的hashCode()方法
         *
         * key是一个对象的引用（可以看成地址）
         * 理论上讲，key的值是否相等，跟计算出的哈希值是否相等，没有必然联系，一切都取决于hashCode()这个方法
         */
        int hash = spread(key.hashCode());

        int binCount = 0;

        Node<K, V>[] tab = table;

        while(true) {
            Node<K, V> f;   // 指向待插入元素应当插入的位置
            int fh;         // 元素f对应的哈希值
            K fk;           // 元素f中的key
            V fv;           // 元素f中的value

            int len;        // 哈希数组容量
            int i;          // 当前key在哈希数组上的索引

            // 如果哈希数组还未初始化，或者容量无效，则需要初始化一个哈希数组
            if(tab == null || (len = tab.length) == 0) {
                // 初始化哈希数组
                tab = initTable();

                /*
                 * 如果哈希数组已经初始化，则需要确定待插入元素所在的哈希槽
                 *
                 * f指向hash所在的哈希槽（链）上的首个元素
                 * f==null意味着需要在哈希槽的相应位置插入首个元素
                 *
                 * 如果当前哈希槽处没有旧元素，说明当前的node将作为首个元素插入
                 */
            } else if((f = tabAt(tab, i = (len - 1) & hash)) == null) {
                // node是相应哈希槽处的首个元素
                Node<K, V> node = new Node<>(hash, key, value);

                // 原子地更新tab[i]为node
                if(casTabAt(tab, i, null, node)) {
                    // 跳出外循环
                    break;                   // no lock when adding to empty bin
                }

                /*
                 * 如果待插入元素所在的哈希槽上已经有别的结点存在，且该结点类型为MOVED
                 * 说明当前哈希数组正在扩容中，此时，可以尝试加速扩容过程
                 */
            } else if((fh = f.hash) == MOVED) {
                tab = helpTransfer(tab, f);

                /*
                 * 如果待插入元素所在的哈希槽上已经有别的结点存在，且当前状态不是在扩容当中，那么首先判断该结点是否为同位元素
                 * 如果遇到了同位元素，但不允许覆盖存储，则直接返回待插入的值
                 */
            } else if(onlyIfAbsent // 如果不允许覆盖存储
                && fh == hash && ((fk = f.key) == key || (fk != null && key.equals(fk))) && (fv = f.val) != null) { // 如果遇到了同位元素

                return fv;

                /*
                 * 如果不是同位元素，则需要搜寻合适的插入位置
                 * 如果是同位元素，且允许覆盖，则直接覆盖旧元素
                 */
            } else {
                V oldVal = null;

                // 对已有元素进行搜索，尝试插入新结点(尾插)
                synchronized(f) {
                    // 如果tab[i]==f，则代表当前待插入状态仍然可信
                    if(tabAt(tab, i) == f) {
                        // 如果当前哈希槽首个元素是普通结点
                        if(fh >= 0) {
                            binCount = 1;

                            // 遍历哈希槽(链)
                            for(Node<K, V> e = f; ; ++binCount) {
                                K ek;

                                // 找到了同位元素，替换旧元素或者跳出循环
                                if(e.hash == hash && ((ek = e.key) == key || (ek != null && key.equals(ek)))) {
                                    // 记录同位元素旧值
                                    oldVal = e.val;

                                    // 如果允许覆盖，则存入新值
                                    if(!onlyIfAbsent) {
                                        e.val = value;
                                    }

                                    // 跳出内循环
                                    break;
                                }

                                Node<K, V> pred = e;
                                e = e.next;

                                // 无法找到同位元素的话，说明需要在哈希槽(链)末尾新增元素
                                if(e==null) {
                                    pred.next = new Node<K, V>(hash, key, value);

                                    // 跳出内循环
                                    break;
                                }
                            } // for

                            // 如果当前哈希槽首个元素是红黑树(头结点)
                        } else if(f instanceof TreeBin) {
                            Node<K, V> p;
                            binCount = 2;
                            if((p = ((TreeBin<K, V>) f).putTreeVal(hash, key, value)) != null) {
                                oldVal = p.val;
                                if(!onlyIfAbsent) {
                                    p.val = value;
                                }
                            }
                        } else if(f instanceof ReservationNode) {
                            throw new IllegalStateException("Recursive update");
                        }
                    }
                } // synchronized

                // 如果对已有元素搜索过，则计数会发生变动，这里需要进一步观察
                if(binCount != 0) {
                    // 哈希槽（链）上的元素数量增加到TREEIFY_THRESHOLD后，这些元素进入波动期，即将从链表转换为红黑树
                    if(binCount >= TREEIFY_THRESHOLD) {
                        treeifyBin(tab, i);
                    }

                    if(oldVal != null) {
                        return oldVal;
                    }

                    break;
                }
            }
        } // while

        // 增加计数
        addCount(1L, binCount);

        return null;
    }
```

### 1.1.2.ConcurrentHashMap在jdk1.8中的设计

**改进一：取消segments字段，直接采用**`transient volatile HashEntry<K,V>[] table`**保存数据，采用table数组元素作为锁，从而实现了对每一行数据进行加锁，进一步减少并发冲突的概率。**

**改进二：将原先table数组＋单向链表的数据结构，变更为table数组＋单向链表＋红黑树的结构。对于hash表来说，最核心的能力在于将key hash之后能均匀的分布在数组中。如果hash之后散列的很均匀，那么table数组中的每个队列长度主要为0或者1。但实际情况并非总是如此理想，虽然ConcurrentHashMap类默认的加载因子为0.75，但是在数据量过大或者运气不佳的情况下，还是会存在一些队列长度过长的情况，如果还是采用单向列表方式，那么查询某个节点的时间复杂度为O\(n\)；因此，对于个数超过8\(默认值\)的列表，jdk1.8中采用了红黑树的结构，那么查询的时间复杂度可以降低到O\(logN\)，可以改进性能。**

为了说明以上2个改动，看一下put操作是如何实现的。

```
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        // 如果table为空，初始化；否则，根据hash值计算得到数组索引i，如果tab[i]为空，直接新建节点Node即可。注：tab[i]实质为链表或者红黑树的首节点。
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        // 如果tab[i]不为空并且hash值为MOVED，说明该链表正在进行transfer操作，返回扩容完成后的table。
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            // 针对首个节点进行加锁操作，而不是segment，进一步减少线程冲突
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            // 如果在链表中找到值为key的节点e，直接设置e.val = value即可。
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            // 如果没有找到值为key的节点，直接新建Node并加入链表即可。
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    // 如果首节点为TreeBin类型，说明为红黑树结构，执行putTreeVal操作。
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                // 如果节点数>＝8，那么转换链表结构为红黑树结构。
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    // 计数增加1，有可能触发transfer操作(扩容)。
    addCount(1L, binCount);
    return null;
}
```

另外，在其他方面也有一些小的改进，比如新增字段 `private transient volatile CounterCell[] counterCells;` 可方便的计算hashmap中所有元素的个数，性能大大优于jdk1.7中的size\(\)方法。

## 1.2.如何实现线程安全的

**jdk1.7**





## 1.3.jdk1.7和jdk1.8的区别

## 1.4.ConcurrentHashMap使用问题

# 2.总结

在ConcurrentHashMap没有出现以前，jdk使用hashtable来实现线程安全，但是hashtable是将整个hash表锁住，所以效率很低下。

针对HashTable会锁整个hash表的问题，ConcurrentHashMap提出了分段锁的解决方案。

分段锁的思想就是：锁的时候不锁整个hash表，而是只锁一部分。

# 3.参考



