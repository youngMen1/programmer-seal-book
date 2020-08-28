# 26 | 数据存储：NoSQL与RDBMS如何取长补短、相辅相成？

近几年，各种非关系型数据库，也就是 NoSQL 发展迅猛，在项目中也非常常见。其中不乏一些使用上的极端情况，比如直接把关系型数据库（RDBMS）全部替换为 NoSQL，或是在不合适的场景下错误地使用 NoSQL。

其实，每种 NoSQL 的特点不同，都有其要着重解决的某一方面的问题。因此，我们在使用 NoSQL 的时候，要尽量让它去处理擅长的场景，否则不但发挥不出它的功能和优势，还可能会导致性能问题。

NoSQL 一般可以分为缓存数据库、时间序列数据库、全文搜索数据库、文档数据库、图数据库等。今天，我会以缓存数据库 Redis、时间序列数据库 InfluxDB、全文搜索数据库 ElasticSearch 为例，通过一些测试案例，和你聊聊这些常见 NoSQL 的特点，以及它们擅长和不擅长的地方。最后，我也还会和你说说 NoSQL 如何与 RDBMS 相辅相成，来构成一套可以应对高并发的复合数据库体系。

## 取长补短之 Redis vs MySQL

Redis 是一款设计简洁的缓存数据库，数据都保存在内存中，所以读写单一 Key 的性能非常高。

我们来做一个简单测试，分别填充 10 万条数据到 Redis 和 MySQL 中。MySQL 中的 name 字段做了索引，相当于 Redis 的 Key，data 字段为 100 字节的数据，相当于 Redis 的 Value：



```

@SpringBootApplication
@Slf4j
public class CommonMistakesApplication {

    //模拟10万条数据存到Redis和MySQL
    public static final int ROWS = 100000;
    public static final String PAYLOAD = IntStream.rangeClosed(1, 100).mapToObj(__ -> "a").collect(Collectors.joining(""));
    @Autowired
    private StringRedisTemplate stringRedisTemplate;
    @Autowired
    private JdbcTemplate jdbcTemplate;
    @Autowired
    private StandardEnvironment standardEnvironment;


    public static void main(String[] args) {
        SpringApplication.run(CommonMistakesApplication.class, args);
    }

    @PostConstruct
    public void init() {
        //使用-Dspring.profiles.active=init启动程序进行初始化
        if (Arrays.stream(standardEnvironment.getActiveProfiles()).anyMatch(s -> s.equalsIgnoreCase("init"))) {
            initRedis();
            initMySQL();
        }
    }

    //填充数据到MySQL
    private void initMySQL() {````
        //删除表
        jdbcTemplate.execute("DROP TABLE IF EXISTS `r`;");
        //新建表，name字段做了索引
        jdbcTemplate.execute("CREATE TABLE `r` (\n" +
                "  `id` bigint(20) NOT NULL AUTO_INCREMENT,\n" +
                "  `data` varchar(2000) NOT NULL,\n" +
                "  `name` varchar(20) NOT NULL,\n" +
                "  PRIMARY KEY (`id`),\n" +
                "  KEY `name` (`name`) USING BTREE\n" +
                ") ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;");

        //批量插入数据
        String sql = "INSERT INTO `r` (`data`,`name`) VALUES (?,?)";
        jdbcTemplate.batchUpdate(sql, new BatchPreparedStatementSetter() {
            @Override
            public void setValues(PreparedStatement preparedStatement, int i) throws SQLException {
                preparedStatement.setString(1, PAYLOAD);
                preparedStatement.setString(2, "item" + i);
            }

            @Override
            public int getBatchSize() {
                return ROWS;
            }
        });
        log.info("init mysql finished with count {}", jdbcTemplate.queryForObject("SELECT COUNT(*) FROM `r`", Long.class));
    }

    //填充数据到Redis
    private void initRedis() {
        IntStream.rangeClosed(1, ROWS).forEach(i -> stringRedisTemplate.opsForValue().set("item" + i, PAYLOAD));
        log.info("init redis finished with count {}", stringRedisTemplate.keys("item*"));
    }
}
```
启动程序后，输出了如下日志，数据全部填充完毕：



```

[14:22:47.195] [main] [INFO ] [o.g.t.c.n.r.CommonMistakesApplication:80  ] - init redis finished with count 100000
[14:22:50.030] [main] [INFO ] [o.g.t.c.n.r.CommonMistakesApplication:74  ] - init mysql finished with count 100000
```

然后，比较一下从 MySQL 和 Redis 随机读取单条数据的性能。“公平”起见，像 Redis 那样，我们使用 MySQL 时也根据 Key 来查 Value，也就是根据 name 字段来查 data 字段，并且我们给 name 字段做了索引：



```

@Autowired
private JdbcTemplate jdbcTemplate;
@Autowired
private StringRedisTemplate stringRedisTemplate;

@GetMapping("redis")
public void redis() {
    //使用随机的Key来查询Value，结果应该等于PAYLOAD
    Assert.assertTrue(stringRedisTemplate.opsForValue().get("item" + (ThreadLocalRandom.current().nextInt(CommonMistakesApplication.ROWS) + 1)).equals(CommonMistakesApplication.PAYLOAD));
}

@GetMapping("mysql")
public void mysql() {
    //根据随机name来查data，name字段有索引，结果应该等于PAYLOAD
    Assert.assertTrue(jdbcTemplate.queryForObject("SELECT data FROM `r` WHERE name=?", new Object[]{("item" + (ThreadLocalRandom.current().nextInt(CommonMistakesApplication.ROWS) + 1))}, String.class)
            .equals(CommonMistakesApplication.PAYLOAD));
}
```

在我的电脑上，使用 wrk 加 10 个线程 50 个并发连接做压测。可以看到，MySQL 90% 的请求需要 61ms，QPS 为 1460；而 Redis 90% 的请求在 5ms 左右，QPS 达到了 14008，几乎是 MySQL 的十倍：

2d289cc94097c2e62aa97a6602d0554e.png

但 Redis 薄弱的地方是，不擅长做 Key 的搜索。对 MySQL，我们可以使用 LIKE 操作前匹配走 B+ 树索引实现快速搜索；但对 Redis，我们使用 Keys 命令对 Key 的搜索，其实相当于在 MySQL 里做全表扫描。

我写一段代码来对比一下性能：



```

@GetMapping("redis2")
public void redis2() {
    Assert.assertTrue(stringRedisTemplate.keys("item71*").size() == 1111);
}
@GetMapping("mysql2")
public void mysql2() {
    Assert.assertTrue(jdbcTemplate.queryForList("SELECT name FROM `r` WHERE name LIKE 'item71%'", String.class).size() == 1111);
}
```

可以看到，在 QPS 方面，**MySQL 的 QPS 达到了 Redis 的 157 倍；在延迟方面，MySQL 的延迟只有 Redis 的十分之一。**


5de7a4a7bf27f8736b0ac09ba0dd1fe8.png

Redis 慢的原因有两个：
* Redis 的 Keys 命令是 O(n) 时间复杂度。如果数据库中 Key 的数量很多，就会非常慢。* * Redis 是单线程的，对于慢的命令如果有并发，串行执行就会非常耗时。

