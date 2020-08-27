# 23 | 缓存设计：缓存可以锦上添花也可以落井下石
通常我们会使用更快的介质（比如内存）作为缓存，来解决较慢介质（比如磁盘）读取数据慢的问题，缓存是用空间换时间，来解决性能问题的一种架构设计模式。更重要的是，磁盘上存储的往往是原始数据，而缓存中保存的可以是面向呈现的数据。这样一来，缓存不仅仅是加快了 IO，还可以减少原始数据的计算工作。

此外，缓存系统一般设计简单，功能相对单一，所以诸如 Redis 这种缓存系统的整体吞吐量，能达到关系型数据库的几倍甚至几十倍，因此缓存特别适用于互联网应用的高并发场景。

使用 Redis 做缓存虽然简单好用，但使用和设计缓存并不是 set 一下这么简单，需要注意缓存的同步、雪崩、并发、穿透等问题。今天，我们就来详细聊聊。

## 不要把 Redis 当作数据库

通常，我们会使用 Redis 等分布式缓存数据库来缓存数据，但是**千万别把 Redis 当做数据库来使用。**我就见过许多案例，因为 Redis 中数据消失导致业务逻辑错误，并且因为没有保留原始数据，业务都无法恢复。

Redis 的确具有数据持久化功能，可以实现服务重启后数据不丢失。这一点，很容易让我们误认为 Redis 可以作为高性能的 KV 数据库。

其实，从本质上来看，Redis（免费版）是一个内存数据库，所有数据保存在内存中，并且直接从内存读写数据响应操作，只不过具有数据持久化能力。所以，Redis 的特点是，处理请求很快，但无法保存超过内存大小的数据。

备注：VM 模式虽然可以保存超过内存大小的数据，但是因为性能原因从 2.6 开始已经被废弃。此外，Redis 企业版提供了 Redis on Flash 可以实现 Key+ 字典 + 热数据保存在内存中，冷数据保存在 SSD 中。

因此，把 Redis 用作缓存，我们需要注意两点。

第一，从客户端的角度来说，缓存数据的特点一定是有原始数据来源，且允许丢失，即使设置的缓存时间是 1 分钟，在 30 秒时缓存数据因为某种原因消失了，我们也要能接受。当数据丢失后，我们需要从原始数据重新加载数据，不能认为缓存系统是绝对可靠的，更不能认为缓存系统不会删除没有过期的数据。

第二，从 Redis 服务端的角度来说，缓存系统可以保存的数据量一定是小于原始数据的。首先，我们应该限制 Redis 对内存的使用量，也就是设置 maxmemory 参数；其次，我们应该根据数据特点，明确 Redis 应该以怎样的算法来驱逐数据。

从Redis 的文档可以看到，常用的数据淘汰策略有：

* allkeys-lru，针对所有 Key，优先删除最近最少使用的 Key；
* volatile-lru，针对带有过期时间的 Key，优先删除最近最少使用的 Key；
* volatile-ttl，针对带有过期时间的 Key，优先删除即将过期的 Key（根据 TTL 的值）；
* allkeys-lfu（Redis 4.0 以上），针对所有 Key，优先删除最少使用的 Key；
* volatile-lfu（Redis 4.0 以上），针对带有过期时间的 Key，优先删除最少使用的 Key。

其实，这些算法是 Key 范围 +Key 选择算法的搭配组合，其中范围有 allkeys 和 volatile 两种，算法有 LRU、TTL 和 LFU 三种。接下来，我就从 Key 范围和算法角度，和你说说如何选择合适的驱逐算法。

首先，从算法角度来说，Redis 4.0 以后推出的 LFU 比 LRU 更“实用”。试想一下，如果一个 Key 访问频率是 1 天一次，但正好在 1 秒前刚访问过，那么 LRU 可能不会选择优先淘汰这个 Key，反而可能会淘汰一个 5 秒访问一次但最近 2 秒没有访问过的 Key，而 LFU 算法不会有这个问题。而 TTL 会比较“头脑简单”一点，优先删除即将过期的 Key，但有可能这个 Key 正在被大量访问。

然后，从 Key 范围角度来说，allkeys 可以确保即使 Key 没有 TTL 也能回收，如果使用的时候客户端总是“忘记”设置缓存的过期时间，那么可以考虑使用这个系列的算法。而 volatile 会更稳妥一些，万一客户端把 Redis 当做了长效缓存使用，只是启动时候初始化一次缓存，那么一旦删除了此类没有 TTL 的数据，可能就会导致客户端出错。

所以，不管是使用者还是管理者都要考虑 Redis 的使用方式，使用者需要考虑应该以缓存的姿势来使用 Redis，管理者应该为 Redis 设置内存限制和合适的驱逐策略，避免出现 OOM。

## 注意缓存雪崩问题

由于缓存系统的 IOPS 比数据库高很多，因此要特别小心短时间内大量缓存失效的情况。这种情况一旦发生，可能就会在瞬间有大量的数据需要回源到数据库查询，对数据库造成极大的压力，极限情况下甚至导致后端数据库直接崩溃。**这就是我们常说的缓存失效，也叫作缓存雪崩。**

从广义上说，产生缓存雪崩的原因有两种：

* 第一种是，缓存系统本身不可用，导致大量请求直接回源到数据库；
* 第二种是，应用设计层面大量的 Key 在同一时间过期，导致大量的数据回源。

第一种原因，主要涉及缓存系统本身高可用的配置，不属于缓存设计层面的问题，所以今天我主要和你说说如何确保大量 Key 不在同一时间被动过期。

程序初始化的时候放入 1000 条城市数据到 Redis 缓存中，过期时间是 30 秒；数据过期后从数据库获取数据然后写入缓存，每次从数据库获取数据后计数器 +1；在程序启动的同时，启动一个定时任务线程每隔一秒输出计数器的值，并把计数器归零。

压测一个随机查询某城市信息的接口，观察一下数据库的 QPS：



```

@Autowired
private StringRedisTemplate stringRedisTemplate;
private AtomicInteger atomicInteger = new AtomicInteger();

@PostConstruct
public void wrongInit() {
    //初始化1000个城市数据到Redis，所有缓存数据有效期30秒
    IntStream.rangeClosed(1, 1000).forEach(i -> stringRedisTemplate.opsForValue().set("city" + i, getCityFromDb(i), 30, TimeUnit.SECONDS));
    log.info("Cache init finished");
    
    //每秒一次，输出数据库访问的QPS
    
   Executors.newSingleThreadScheduledExecutor().scheduleAtFixedRate(() -> {
        log.info("DB QPS : {}", atomicInteger.getAndSet(0));
    }, 0, 1, TimeUnit.SECONDS);
}

@GetMapping("city")
public String city() {
    //随机查询一个城市
    int id = ThreadLocalRandom.current().nextInt(1000) + 1;
    String key = "city" + id;
    String data = stringRedisTemplate.opsForValue().get(key);
    if (data == null) {
        //回源到数据库查询
        data = getCityFromDb(id);
        if (!StringUtils.isEmpty(data))
            //缓存30秒过期
            stringRedisTemplate.opsForValue().set(key, data, 30, TimeUnit.SECONDS);
    }
    return data;
}

private String getCityFromDb(int cityId) {
    //模拟查询数据库，查一次增加计数器加一
    atomicInteger.incrementAndGet();
    return "citydata" + System.currentTimeMillis();
}
```

使用 **wrk 工具**，设置 10 线程 10 连接压测 city 接口：



```
wrk -c10 -t10 -d 100s http://localhost:45678/cacheinvalid/city
```

启动程序 30 秒后缓存过期，回源的数据库 QPS 最高达到了 700 多：

918a91e34725e475cdee746d5ba8aa6b.png


解决缓存 Key 同时大规模失效需要回源，导致数据库压力激增问题的方式有两种。

方案一，差异化缓存过期时间，不要让大量的 Key 在同一时间过期。比如，在初始化缓存的时候，设置缓存的过期时间是 30 秒 +10 秒以内的随机延迟（扰动值）。这样，这些 Key 不会集中在 30 秒这个时刻过期，而是会分散在 30~40 秒之间过期：


```

@PostConstruct
public void rightInit1() {
    //这次缓存的过期时间是30秒+10秒内的随机延迟
    IntStream.rangeClosed(1, 1000).forEach(i -> stringRedisTemplate.opsForValue().set("city" + i, getCityFromDb(i), 30 + ThreadLocalRandom.current().nextInt(10), TimeUnit.SECONDS));
    log.info("Cache init finished");
    //同样1秒一次输出数据库QPS：
   Executors.newSingleThreadScheduledExecutor().scheduleAtFixedRate(() -> {
        log.info("DB QPS : {}", atomicInteger.getAndSet(0));
    }, 0, 1, TimeUnit.SECONDS);
}
```
修改后，缓存过期时的回源不会集中在同一秒，数据库的 QPS 从 700 多降到了最高 100 左右：

6f4a666cf48c4d1373aead40afb57a35.png

方案二，让缓存不主动过期。初始化缓存数据的时候设置缓存永不过期，然后启动一个后台线程 30 秒一次定时把所有数据更新到缓存，而且通过适当的休眠，控制从数据库更新数据的频率，降低数据库压力：



```

@PostConstruct
public void rightInit2() throws InterruptedException {
    CountDownLatch countDownLatch = new CountDownLatch(1);
    //每隔30秒全量更新一次缓存 
    Executors.newSingleThreadScheduledExecutor().scheduleAtFixedRate(() -> {
        IntStream.rangeClosed(1, 1000).forEach(i -> {
            String data = getCityFromDb(i);
            //模拟更新缓存需要一定的时间
            try {
                TimeUnit.MILLISECONDS.sleep(20);
            } catch (InterruptedException e) { }
            if (!StringUtils.isEmpty(data)) {
                //缓存永不过期，被动更新
                stringRedisTemplate.opsForValue().set("city" + i, data);
            }
        });
        log.info("Cache update finished");
        //启动程序的时候需要等待首次更新缓存完成
        countDownLatch.countDown();
    }, 0, 30, TimeUnit.SECONDS);

    Executors.newSingleThreadScheduledExecutor().scheduleAtFixedRate(() -> {
        log.info("DB QPS : {}", atomicInteger.getAndSet(0));
    }, 0, 1, TimeUnit.SECONDS);

    countDownLatch.await();
}
```

这样，可以把回源到数据库的并发限制在 1：
.
63ccde3fdf058b48431fc7c554fed828.png

在真实的业务场景下，不一定要这么严格地使用双重检查分布式锁进行全局的并发限制，因为这样虽然可以把数据库回源并发降到最低，但也限制了缓存失效时的并发。可以考虑的方式是：

* 方案一，使用进程内的锁进行限制，这样每一个节点都可以以一个并发回源数据库；

* 方案二，不使用锁进行限制，而是使用类似 Semaphore 的工具限制并发数，比如限制为 10，这样既限制了回源并发数不至于太大，又能使得一定量的线程可以同时回源。

## 注意缓存穿透问题

在之前的例子中，缓存回源的逻辑都是当缓存中查不到需要的数据时，回源到数据库查询。这里容易出现的一个漏洞是，缓存中没有数据不一定代表数据没有缓存，还有一种可能是原始数据压根就不存在。

比如下面的例子。数据库中只保存有 ID 介于 0（不含）和 10000（包含）之间的用户，如果从数据库查询 ID 不在这个区间的用户，会得到空字符串，所以缓存中缓存的也是空字符串。如果使用 ID=0 去压接口的话，从缓存中查出了空字符串，认为是缓存中没有数据回源查询，其实相当于每次都回源：


```

@GetMapping("wrong")
public String wrong(@RequestParam("id") int id) {
    String key = "user" + id;
    String data = stringRedisTemplate.opsForValue().get(key);
    //无法区分是无效用户还是缓存失效
    if (StringUtils.isEmpty(data)) {
        data = getCityFromDb(id);
        stringRedisTemplate.opsForValue().set(key, data, 30, TimeUnit.SECONDS);
    }
    return data;
}

private String getCityFromDb(int id) {
    atomicInteger.incrementAndGet();
    //注意，只有ID介于0（不含）和10000（包含）之间的用户才是有效用户，可以查询到用户信息
    if (id > 0 && id <= 10000) return "userdata";
    //否则返回空字符串
    return "";
}
```

压测后数据库的 QPS 达到了几千：

dc2ee3259dd21d55a845dc4a8b9146d2.png

如果这种漏洞被恶意利用的话，就会对数据库造成很大的性能压力。**这就是缓存穿透。**

这里需要注意，缓存穿透和缓存击穿的区别：

* 缓存穿透是指，缓存没有起到压力缓冲的作用；

* 而缓存击穿是指，缓存失效时瞬时的并发打到数据库。

**解决缓存穿透有以下两种方案。**

方案一，对于不存在的数据，同样设置一个特殊的 Value 到缓存中，比如当数据库中查出的用户信息为空的时候，设置 NODATA 这样具有特殊含义的字符串到缓存中。这样下次请求缓存的时候还是可以命中缓存，即直接从缓存返回结果，不查询数据库：




```

@GetMapping("right")
public String right(@RequestParam("id") int id) {
    String key = "user" + id;
    String data = stringRedisTemplate.opsForValue().get(key);
    if (StringUtils.isEmpty(data)) {
        data = getCityFromDb(id);
        //校验从数据库返回的数据是否有效
        if (!StringUtils.isEmpty(data)) {
            stringRedisTemplate.opsForValue().set(key, data, 30, TimeUnit.SECONDS);
        }
        else {
            //如果无效，直接在缓存中设置一个NODATA，这样下次查询时即使是无效用户还是可以命中缓存
            stringRedisTemplate.opsForValue().set(key, "NODATA", 30, TimeUnit.SECONDS);
        }
    }
    return data;
}
```

但，这种方式可能会把大量无效的数据加入缓存中，如果担心大量无效数据占满缓存的话还可以考虑方案二，即使用布隆过滤器做前置过滤。

布隆过滤器是一种概率型数据库结构，由一个很长的二进制向量和一系列随机映射函数组成。它的原理是，当一个元素被加入集合时，通过 k 个散列函数将这个元素映射成一个 m 位 bit 数组中的 k 个点，并置为 1。

检索时，我们只要看看这些点是不是都是 1 就（大概）知道集合中有没有它了。如果这些点有任何一个 0，则被检元素一定不在；如果都是 1，则被检元素很可能在。原理如下图所示：

c58cb0c65c37f4c1bf3aceba1c00d71f.png