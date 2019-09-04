# Memcached 和 Redis 分布式锁方案

分布式缓存，能解决单台服务器内存不能无限扩张的瓶颈。在分布式缓存的应用中，会遇到多个客户端同时争用的问题。这个时候，需要用到分布式锁，得到锁的客户端才有操作权限。

Memcached 和 Redis 是常用的分布式缓存构建方案，下面列举下基于Memcached 和 Redis 分布式锁的实现方法。

**Memcached 分布式锁**

**Memcached** 可以使用 **add** 命令，该命令只有KEY**不存在**时，才进行添加，或者不会处理。Memcached 所有命令都是原子性的，并发下add 同一个KEY ，只会一个会成功。

利用这个原理，可以先定义一个 锁 LockKEY ，add 成功的认为是得到锁。并且设置\[**过期超时**\] 时间，保证**宕机**后，也不会**死锁**。

在具体操作完后，判断是否此次操作已超时。如果超时则不删除锁，如果不超时则删除锁。

伪代码：

```
 1          if (mc.Add("LockKey", "Value", expiredtime))
 2             {
 3                 //得到锁
 4                 try
 5                 {
 6                     //do business  function
 7 
 8                     //检查超时
 9                     if (!CheckedTimeOut())
10                     {
11                         mc.Delete("LockKey");
12                     }
13                 }
14                 catch (Exception e)
15                 {
16                     mc.Delete("LockKey");
17                 }
18                
19             }
```

**Redis 分布式锁**

**Redis  **没有add 命令，但有**SETNX**（SET if Not eXists）若给定的 key 已经存在，则 SETNX不做任何动作。设置成功，返回 1 。设置失败，返回 0 。

SETNX 命令不能设置过期时间，需要再使用 EXPIRE 命令设置过期时间。

伪代码：

