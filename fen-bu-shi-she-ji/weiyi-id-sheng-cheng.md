## 唯一ID 生成

### 全局唯一ID

* [《高并发分布式系统中生成全局唯一Id汇总》](https://blog.csdn.net/hemin1003/article/details/80921615)

  * Twitter 方案（Snowflake 算法）：41位时间戳+10位机器标识（比如IP，服务器名称等）+12位序列号\(本地计数器\)
  * Flicker 方案：MySQL自增ID + "REPLACE INTO XXX:SELECT LAST\_INSERT\_ID\(\);"
  * UUID：缺点，无序，字符串过长，占用空间，影响检索性能。
  * MongoDB 方案：利用 ObjectId。缺点：不能自增。

* [《TDDL 在分布式下的SEQUENCE原理》](https://blog.csdn.net/hdu09075340/article/details/79103851)

  * 在数据库中创建 sequence 表，用于记录，当前已被占用的id最大值。
  * 每台客户端主机取一个id区间（比如 1000~2000）缓存在本地，并更新 sequence 表中的id最大值记录。
  * 客户端主机之间取不同的id区间，用完再取，使用乐观锁机制控制并发。



