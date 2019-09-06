一致性哈希（Consistent hashing）算法是由 MIT 的Karger 等人与1997年在一篇学术论文（《Consistent hashing and random trees: distributed caching protocols for relieving hot spots on the World Wide Web》）中提出来的，用于解决分布式缓存数据分布问题。在传统的哈希算法下，每条缓存数据落在那个节点是通过哈希算法和服务器节点数量计算出来的，一旦服务器节点数量发生增加或者介绍，哈希值需要重新计算，此时几乎所有的数据和服务器节点的对应关系也会随之发生变化，进而会造成绝大多数缓存的失效。一致性哈希算法通过环形结构和虚拟节点的概念，确保了在缓存服务器节点数量发生变化时大部分数据保持原地不动，从而大幅提高了缓存的有效性。下面我们通过例子来解释一致性哈希的原理。

​ 比如有 n 个节点，对于缓存 数据（k，v）具体存在哪个节点往往 hash\(k\) % n 来计算处理，举一个例子如下表所示，一共有个3个节点，hash函数采用 md5 。

| 缓存key | hash\(k\) % n | 服务器节点 |
| :--- | :--- | :--- |
| user\_nick\_rommel | 1 | 192.168.56.101 |
| user\_nick\_pandy | 0 | 192.168.56.100 |
| user\_nick\_sam | 2 | 192.168.56.102 |

> Md5 的计算结果一般是一串32位的16进制字符串，做取模运算时原始数字较长，实际使用时，可以只截取最后4位或者8位使用，因为hash函数具有随机性，当数据量足球大时，截取部分数据也能保证数据的均匀分布。比如 md5\('user\_nick\_rommel'\)，对应字符串为 29e4fd2a0f05bd63343ae2276ca5038e，取最后4位`038e`转成10进制整数在进行取模运算，038e 对应的10进制数为 910，取模计算得 1 \(9102 % 3 = 1\)。

​ 如果此时对缓存服务器进行扩容，添加一个新节点如 192.168.56.103，那么按照上面的计算方式，n 变为4， 得到的结果如下：

| 缓存key | hash\(k\) % n | 服务器节点 |
| :--- | :--- | :--- |
| user\_nick\_rommel | 2 | 192.168.56.102 |
| user\_nick\_pandy | 2 | 192.168.56.102 |
| user\_nick\_sam | 3 | 192.168.56.103 |

​ 从结果中可见，缓存对应关系完全发生改变，比如 user\_nick\_rommel 这个可以，添加节点钱可以从 192.168.56.101 中读到，添加节点后却读不到了，。一般缓存失效时应用程序都会重新从后端服务加载数据（比如数据库），以这种这种方式分配缓存，当缓节点数量发生改变时，会造成大面积的缓存失效，这回造成后端服务瞬间压力上升，压力过大会造成服务不可以用，如果服务出于关键节点，甚至还会引发雪崩效应（TODO）。

​ 在实际应用中，缓存节点由于故障挂掉，或者空间不足而进行扩容，缓存节点的增减是比较常见的事情，但上面传统方式会使服务的不可靠，下面看下一致性哈希是如何解决这个问题的。

​ 在一致性哈希算法中，首先将哈希空间映射到一个虚拟的环上，环上的数值分从 0 到 2^32-1（哈希值的范围），如下图：

![img](/static/image/D2CFEC7D-5ACB-49C3-B67F-12BA52254454.png)

在一致性哈希算法刚提出来的时候，32位系统还是主流，2^32-1 相当于最大Integer，现在的应用服务器普遍都是64位系统，在使用使用一致性哈希算法时可以根据实际情况适当变通，比如将哈希值空间放大到 2^64-1。

​ 然后使用同样的哈希算法将缓存服务节点（通常通过服务器IP+端口作为节点的key）和数据键映射到环上的位置。再决定数据落在那台服务器上时，使用一致的方向（比如顺时针方向）沿环查找，遇到的第一个有效服务器就是缓存保存的地方，如图：

![img](/static/image/7D4C84D6-8843-42FC-AAB4-C5D5EA308C1B.png)

当有新的服务器节点加入时，按照同样的哈希算法将新节点映射到环上的某处位置，和新节点相邻的数据逆时针节点会进行迁移，其他节点保持不变。如下图，当新加入一台新服务器 192.168.56.104 时，user\_nick\_pandy 这条缓存数据的请求根据算法会落在192.168.56.104 这台及其上，其他节点不受任何影响。

![img](/static/image/A026DF6E-060A-4295-83CB-FC4A7D229A03.png)

另外，由于哈希计算的记过通常都比较随机，如果缓存服务器比较少的话，可能会出现数据分配冷热不均的问题，如下图所示，大部分数据都会存储在 node3 节点上。

![img](/static/image/26145228-2DA8-4168-BBE9-DA99F4BD0016.png)

​ 为了解决这个问题，我们引入虚拟节点的概念，在实体服务器不增加的情况下，用多个虚拟节点替代原来的单个实体节点，一台服务服务器在环上就对应多个位置，这样可以让数据存储更加均匀，各服务器的负载页更加平衡，如图：

9B7CB81A-434E-4C33-B65E-3821C19E3B70.png

使用 Java 代码实现一致性哈希的例子如下：

```
import java.util.SortedMap;
import java.util.TreeMap;
import org.apache.commons.codec.digest.DigestUtils;

public class ConsistentHashingTest {

    // 真实缓存地址
    private final String[] cacheServers = { "192.168.56.101:11211", "192.168.56.102:11211", "192.168.56.103:11211" };

      // 保存虚拟节点
    private final SortedMap<Long, String> nodes = new TreeMap<Long, String>();

    // 每个虚拟节点的数量
    private final int VIRTUAL_NODE_NUM = 3;

    public ConsistentHashingTest(){
       //初始化
       for(String eachServer : cacheServers) {
           this.addNode(eachServer);
       }
    }

    // 创建虚拟节点
    public void addNode(String nodeKey) {
       //为每一个实体节点创建3个虚拟节点
       for (int i = 0; i < VIRTUAL_NODE_NUM; i++) {
           long eachHashCode = this.hash(nodeKey + ":" + i);
           nodes.put(eachHashCode, nodeKey);
       }
    }

    // hash 函数 可以使用 md5, sha-1, sha-256 等
    // 虽然 md5, sha-1 哈希算法在签名领域已经不再安全，但运算速度比较快，在非安全领域是可以使用的。
    // DigestUtils 是来自于 apache 中的 org.apache.commons.codec.digest 中的工具类
    private long hash(String key) {
       Stringmd5key = DigestUtils.md5Hex(key);
       return Long.parseLong(md5key.substring(0, 15), 16) % ((long) Math.pow(2, 32));
    }

    //按照同一个方向寻找
    public String getRealServer(String key) {
       long hashCode = this.hash(key);
       SortedMap<Long,String> tailMap =
              nodes.tailMap(hashCode);
       long serverKey = tailMap.isEmpty() ? nodes.firstKey() : tailMap.firstKey(); 
       return nodes.get(serverKey);
    }

    public static void main(String[] args) {
       ConsistentHashingTestt = new ConsistentHashingTest();
       System.out.println(t.getRealServer("my-cache-key"));
    }
}
```

另外，前文提到的（TODO 章节）Guava Cache 框架支持一致性哈希，实例代码如下：

```
//实体缓存服务器
String[]cacheServers = { "192.168.56.101:11211", "192.168.56.102:11211", "192.168.56.103:11211" };

// 缓存数据的key
String key = "my-test-cache-key";

// 计算缓存 key 对应的 hash 值，这里使用 MurmurHash 算法，MurmurHash 是一种高性能低碰撞的算法。此外，还支持  md5、sha1/sha256/sha512、orc32、adler32 等哈希算法。 
HashCode hashCode = Hashing.murmur3_32().newHasher().putString(key, Charsets.UTF_8).hash();

// 通过一致性哈希方式计算，缓存key对应的服务器主机是那一台，bucket 的范围在 0 ~ cacheServers.length -1
int bucket = Hashing.consistentHash(hashCode, cacheServers.length);
```

![img](/static/image/D2CFEC7D-5ACB-49C3-B67F-12BA52254454 (1).png)

