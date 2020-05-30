# 1.[MYSQL分页limit速度太慢优化方法](https://www.cnblogs.com/shiwenhu/p/5757250.html)

在[mysql](http://www.php-z.com/)中limit可以实现快速分页，但是如果数据到了几百万时我们的limit必须优化才能有效的合理的实现分页了，否则可能卡死你的服务器哦。

当一个表数据有几百万的数据的时候成了问题！

如 select \* from table limit 0,10 这个没有问题 当 limit 200000,10 的时候数据读取就很慢，可以按照一下方法解决  
第一页会很快

**PERCONA PERFORMANCE CONFERENCE 2009上，来自雅虎的几位工程师带来了一篇”EfficientPagination Using MySQL”的报告**

**limit10000,20的意思扫描满足条件的10020行，扔掉前面的10000行，返回最后的20行，问题就在这里。**

**LIMIT 451350 , 30 扫描了45万多行，怪不得慢的都堵死了。**

但是

limit 30 这样的语句仅仅扫描30行。

**那么如果我们之前记录了最大ID，就可以在这里做文章**

**举个例子**

日常分页SQL语句

```
select id,name,content from users order by id asc limit 100000,20
```

扫描100020行

**如果记录了上次的最大ID**

select id,name,content from users where id&gt;100073 order by id asc limit 20  
扫描20行。

总数据有500万左右

以下例子 当时候 select \* from wl\_tagindex where byname=’f’ order by id limit 300000,10 执行时间是 3.21s

优化后：

```
select * from (
select id from wl_tagindex
where byname=’f’ order by id limit 300000,10
) a
left join wl_tagindex b on a.id=b.id
```

执行时间为 0.11s 速度明显提升

**这里需要说明的是 我这里用到的字段是 byname ,id 需要把这两个字段做复合索引，否则的话效果提升不明显**

# 2.总结

当一个数据库表过于庞大，LIMIT offset, length中的offset值过大，则SQL查询语句会非常缓慢，你需增加order by，**并且order by字段需要建立索引。**  
如果使用子查询去优化LIMIT的话，则子查询必须是连续的，某种意义来讲，子查询不应该有where条件，where会过滤数据，使数据失去连续性。  
如果你查询的记录比较大，并且数据传输量比较大，比如包含了text类型的field，则可以通过建立子查询。

```
SELECT id,title,content FROM items WHERE id IN (SELECT id FROM items ORDER BY id limit 900000, 10);
```

如果limit语句的offset较大，你可以通过传递pk键值来减小offset = 0，这个主键最好是int类型并且auto\_increment

```
SELECT * FROM users WHERE uid > 456891 ORDER BY uid LIMIT 0, 10;
```

这条语句，大意如下:

```
SELECT * FROM users WHERE uid >= (SELECT uid FROM users ORDER BY uid limit 895682, 1) limit 0, 10;
```

如果limit的offset值过大，用户也会翻页疲劳，你可以设置一个offset最大的，超过了可以另行处理，一般连续翻页过大，用户体验很差，则应该提供更优的用户体验给用户。

### 2.1.limit 分页优化方法

#### 2.1.子查询优化法

  
先找出第一条数据，然后大于等于这条数据的id就是要获取的数据  
缺点：数据必须是连续的，可以说不能有where条件，where条件会筛选数据，导致数据失去连续性

实验下

mysql&gt; set profi=1;  
Query OK, 0 rows affected \(0.00 sec\)

mysql&gt; select count\(\*\) from Member;  
+———-+  
\| count\(\*\) \|  
+———-+  
\| 169566 \|  
+———-+  
1 row in set \(0.00 sec\)

mysql&gt; pager grep !~-  
PAGER set to ‘grep !~-‘

mysql&gt; select \* from Member limit 10, 100;  
100 rows in set \(0.00 sec\)

mysql&gt; select \* from Member where MemberID &gt;= \(select MemberID from Member limit 10,1\) limit 100;  
100 rows in set \(0.00 sec\)

mysql&gt; select \* from Member limit 1000, 100;  
100 rows in set \(0.01 sec\)

mysql&gt; select \* from Member where MemberID &gt;= \(select MemberID from Member limit 1000,1\) limit 100;  
100 rows in set \(0.00 sec\)

mysql&gt; select \* from Member limit 100000, 100;  
100 rows in set \(0.10 sec\)

mysql&gt; select \* from Member where MemberID &gt;= \(select MemberID from Member limit 100000,1\) limit 100;  
100 rows in set \(0.02 sec\)

mysql&gt; nopager  
PAGER set to stdout

mysql&gt; show profilesG  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 1. row \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
Query\_ID: 1  
Duration: 0.00003300  
Query: select count\(\*\) from Member

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 2. row \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
Query\_ID: 2  
Duration: 0.00167000  
Query: select \* from Member limit 10, 100  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 3. row \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
Query\_ID: 3  
Duration: 0.00112400  
Query: select \* from Member where MemberID &gt;= \(select MemberID from Member limit 10,1\) limit 100

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 4. row \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
Query\_ID: 4  
Duration: 0.00263200  
Query: select \* from Member limit 1000, 100  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 5. row \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
Query\_ID: 5  
Duration: 0.00134000  
Query: select \* from Member where MemberID &gt;= \(select MemberID from Member limit 1000,1\) limit 100

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 6. row \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
Query\_ID: 6  
Duration: 0.09956700  
Query: select \* from Member limit 100000, 100  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 7. row \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
Query\_ID: 7  
Duration: 0.02447700  
Query: select \* from Member where MemberID &gt;= \(select MemberID from Member limit 100000,1\) limit 100

从结果中可以得知，当偏移1000以上使用子查询法可以有效的提高性能。

2.倒排表优化法  
倒排表法类似建立索引，用一张表来维护页数，然后通过高效的连接得到数据

缺点：只适合数据数固定的情况，数据不能删除，维护页表困难

3.反向查找优化法  
当偏移超过一半记录数的时候，先用排序，这样偏移就反转了

缺点：order by优化比较麻烦，要增加索引，索引影响数据的修改效率，并且要知道总记录数  
，偏移大于数据的一半

引用  
limit偏移算法：  
正向查找： \(当前页 – 1\) \* 页长度  
反向查找： 总记录 – 当前页 \* 页长度

做下实验，看看性能如何

总记录数：1,628,775  
每页记录数： 40  
总页数：1,628,775 / 40 = 40720  
中间页数：40720 / 2 = 20360

第21000页  
正向查找SQL:  
Sql代码  
SELECT \* FROM \`abc\` WHERE \`BatchID\` = 123 LIMIT 839960, 40  
时间：1.8696 秒

反向查找sql:  
Sql代码  
SELECT \* FROM \`abc\` WHERE \`BatchID\` = 123 ORDER BY InputDate DESC LIMIT 788775, 40  
时间：1.8336 秒

第30000页  
正向查找SQL:  
Sql代码

1.SELECT \* FROM \`abc\` WHERE \`BatchID\` = 123 LIMIT 1199960, 40  
SELECT \* FROM \`abc\` WHERE \`BatchID\` = 123 LIMIT 1199960, 40

时间：2.6493 秒

反向查找sql:  
Sql代码  
1.SELECT \* FROM \`abc\` WHERE \`BatchID\` = 123 ORDER BY InputDate DESC LIMIT 428775, 40  
SELECT \* FROM \`abc\` WHERE \`BatchID\` = 123 ORDER BY InputDate DESC LIMIT 428775, 40

时间：1.0035 秒

注意，反向查找的结果是是降序desc的，并且InputDate是记录的插入时间，也可以用主键联合索引，但是不方便。

4.limit限制优化法  
把limit偏移量限制低于某个数。。超过这个数等于没数据，我记得alibaba的dba说过他们是这样做的

5.只查索引法

# 2.参考

[MYSQL分页limit速度太慢优化方法](https://www.cnblogs.com/shiwenhu/p/5757250.html)：

[https://www.cnblogs.com/shiwenhu/p/5757250.html](https://www.cnblogs.com/shiwenhu/p/5757250.html)

