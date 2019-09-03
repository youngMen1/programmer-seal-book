# [Mysql查询语句使用select.. for update导致的数据库死锁分析](https://www.cnblogs.com/Lawson/p/5008741.html)

近期有一个业务需求，多台机器需要同时从Mysql一个表里查询数据并做后续业务逻辑，为了防止多台机器同时拿到一样的数据，每台机器需要在获取时锁住获取数据的数据段，保证多台机器不拿到相同的数据。

我们Mysql的存储引擎是innodb，支持行锁。解决同时拿数据的方法有很多，为了更加简单，不增加其他表和服务的情况下，我们考虑采用select... for update的方式，这样X锁锁住查询的数据段，表里其他数据没有锁，其他业务逻辑还是可以操作。

这样一台服务器比如select .. for update limit 0,30时，其他服务器执行同样sql语句会自动等待释放锁，等待前一台服务器锁释放后，该台服务器就能查询下一个30条数据。如果要求更智能，oracle支持for update skip locked跳过锁区域，这样能不等待马上查询没有被锁住的下一个30条记录。

下面说下mysql for update导致的死锁。

经过分析，mysql的innodb存储引擎实务锁虽然是锁行，但它内部是锁索引的，根据where条件和select的值是否只有主键或非主键索引来判断怎么锁，比如只有主键，则锁主键索引，如果只有非主键，则锁非主键索引，如果主键非主键都有，则内部会按照顺序锁。但同样的select .. for update语句怎么就死锁了呢？同样的sql语句查询条件和结果顺序都一致，按理不会导致一个锁了主键索引，等待锁非主键索引，另外一个锁了非主键索引，等待主键索引导致的死锁。

最后经过分析，我们项目里发现是for update的sql语句，和另外一个update非select数据的sql语句导致的死锁。

比如有60条数据，select .. for update查询第31-60条数据，update在更新1-10条数据，按照innodb存储引擎的行锁原理，应该不会导致不同行的锁导致的互相等待。开始以为是行锁在数据量较大情况下，会锁数据块。导致一个段的数据被锁住，但经过大量数据测试，发现感觉把整个表都锁住了，但实际不是。

下面举几个例子说明：

数据从id =400000的数据开始，IsSuccess和GetTime字段都为0，现在如果400000数据的IsSuccess为1了。执行下面两条sql.

```
-- 1:
set autocommit=0;
begin;
select * from table1 where getTime < 1 and IsSuccess=0 order by id asc limit 0,30 for update;
commit;
-- 2:
update table1 a set IsSuccess=0 where id =400000;
```



