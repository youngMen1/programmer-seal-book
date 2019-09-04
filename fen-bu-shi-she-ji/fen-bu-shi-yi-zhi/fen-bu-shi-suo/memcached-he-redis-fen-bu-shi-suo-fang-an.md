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

**Redis  **没有add 命令，但有**SETNX**（SET if Not eXists）若给定的 key 已经存在，则 SETNX不做任何动作。设置成功，返回 1 。设置失败，返回 0 。

SETNX 命令不能设置过期时间，需要再使用 EXPIRE 命令设置过期时间。

伪代码：

```
int lockResult = rd.SETNX("LockKey", "Value");
if (lockResult == 1)
{
    //[1]得到锁

    //[2]设置超时过期时间
    rd.EXPIRE("LockKey", expiredtime);

    try
    {
        //do business  function

        //检查超时
        if (!CheckedTimeOut())
        {
            rd.DEL("LockKey");
        }
    }
    catch (Exception e)
    {
        rd.DEL("LockKey");
    }

}
```

这种做法，有一个很大的潜在风险。\[1\]得到锁后，再执行\[2\] 设置过期时间。如果在这期间出现宕机，则会导致没有设置过期时间。按Redis 的默认缓存过期策略，这个锁将不会释放，产生死锁。

所以不推荐用这种做法，应该用其它方式来实现锁的超时过期策略：

```
 1：**SETNX  **value 值=当前时间+过期超时时间，返回1 则获得锁，返回0则没有获得锁。转2。

 2：**GET** 获取 value 的值 。判断锁是否过期超时。如果超时，转3。

 3：**GETSET（**将给定 key 的值设为 value ，并返回 key 的旧值**），GETSET ** value 值=当前时间+过期超时时间， 判断得到的value 如果仍然是超时的，那就说明得到锁，否则没有得到锁。
```

从2并发进到3 的操作，会多次改写超时时间，但这个不会有什么影响。

伪代码：

```
string expiredtime = DateTime.Now.AddMinutes(LockTimeoutMinutes).ToString();
int lockResult = rd.SETNX("LockKey", expiredtime);
bool getLock = false;
if (lockResult == 1)
{
    //得到锁
    getLock = true;
}
else
{
    string curExpiredtime = rd.GET("LockKey");
 
    //检查锁超时
    if (CheckedLockTimeOut(expiredtime))
    {
        expiredtime = DateTime.Now.AddMinutes(LockTimeoutMinutes).ToString();
        string newExpiredTime = GETSET(expiredtime);
        if (CheckedLockTimeOut(newExpiredTime))
        {
            //得到锁
            getLock = true;
        }
    }
}
if (getLock)
{
    try
    {
        //do business  function
 
        //检查超时
        if (!CheckedTimeOut())
        {
            rd.DEL("LockKey");
        }
    }
    catch (Exception e)
    {
        rd.DEL("LockKey");
    }
}
```



