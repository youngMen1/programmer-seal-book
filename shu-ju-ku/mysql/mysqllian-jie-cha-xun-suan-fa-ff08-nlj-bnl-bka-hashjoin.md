# 1.MySQL联接查询算法（NLJ、BNL、BKA、HashJoin）

## 1.1.联接过程介绍

为了后面一些测试案例，我们事先创建了两张表，表数据如下：

```
CREATE TABLE t1 (m1 int, n1 char(1));
CREATE TABLE t2 (m2 int, n2 char(1));
INSERT INTO t1 VALUES(1, 'a'), (2, 'b'), (3, 'c');
INSERT INTO t2 VALUES(2, 'b'), (3, 'c'), (4, 'd'), (5, 'e'), (6, 'f');
```

联接操作的本质就是把各个联接表中的记录都取出来依次匹配的组合加入结果集并返回给用户。如果没有任何限制条件的话，多表联接起来产生的笛卡尔积可能是非常巨大的。比方说3个100行记录的表连接起来产生的笛卡尔积就有100×100×100=1000000行数据！所以在连接的时候过滤掉特定记录组合是有必要的，在连接查询中的过滤条件可以分成两种，我们以一个JOIN查询为例：

```
SELECT * FROM t1, t2 WHERE t1.m1 > 1 AND t1.m1 = t2.m2 AND t2.n2 < 'd';
```

* 涉及单表的条件

```
WHERE条件也可以称为搜索条件，比如t1.m1 > 1是只针对t1表的过滤条件，t2.n2 < ‘d’是只针对t2表的过滤条件。
```

* 涉及两表的条件

比如t1.m1 = t2.m2、t1.n1 &gt; t2.n2等，这些条件中涉及到了两个表，我们稍后会仔细分析这种过滤条件是如何使用的。  
在这个查询中我们指明了这三个过滤条件：

1. t1.m1 &gt; 1

2. t1.m1 = t2.m2

3. t2.n2 &lt; ‘d’

那么这个连接查询的大致执行过程如下：

首先确定第一个需要查询的表，这个表称之为驱动表。怎样在单表中执行查询语句，只需要选取代价最小的那种访问方法去执行单表查询语句就好了（就是说从const、ref、ref\_or\_null、range、index、all这些执行方法中选取代价最小的去执行查询）。此处假设使用t1作为驱动表，那么就需要到t1表中找满足t1.m1&gt;

1的记录，假设这里并没有给t1字段添加索引，所以此处查询t1表的访问方法就设定为all吧，也就是采用全表扫描的方式执行单表查询。关于如何提升连接查询的性能我们之后再说，现在先把基本概念捋清楚哈。所以查询过程就如下图所示：

![](/static/image/2019010408114680.jpg)

针对上一步骤中从驱动表产生的结果集中的每一条记录，分别需要到t2表中查找匹配的记录，所谓匹配的记录，指的是符合过滤条件的记录。因为是根据t1表中的记录去找t2表中的记录，所以t2表也可以被称之为被驱动表。比如上一步骤从驱动表中得到了2条记录，所以需要查询2次t2表。此时涉及两个表的列的过滤条件t1.m1 = t2.m2就派上用场了：

* 当t1.m1 = 2时，过滤条件t1.m1 = t2.m2就相当于t2.m2 = 2，所以此时t2表相当于有了t1.m1 = 2、t2.n2 &lt; ‘d’这两个过滤条件，然后到t2表中执行单表查询。

* 当t1.m1 = 3时，过滤条件t1.m1 = t2.m2就相当于t2.m2 = 3，所以此时t2表相当于有了t1.m1 = 3、t2.n2 &lt; ‘d’这两个过滤条件，然后到t2表中执行单表查询。

所以整个连接查询的执行过程就如下图所示：

![](/static/image/2019010408125195-1000x401.jpg)

也就是说整个连接查询最后的结果只有两条符合过滤条件的记录：

```
+------+------+------+------+
| m1   | n1   | m2   | n2   |
+------+------+------+------+
|    2 | b    |    2 | b    |
|    3 | c    |    3 | c    |
+------+------+------+------+
```

从上边两个步骤可以看出来，我们上边说的这个两表联接查询共需要查询1次t1表，2次t2表。当然这是在特定的过滤条件下的结果，如果我们把t1.m1 &gt; 1这个条件去掉，那么从t1表中查出的记录就有3条，就需要查询3次t3表了。也就是说在两表连接查询中，驱动表只需要访问一次，被驱动表可能被访问多次，这种方式在MySQL中有一个专有名词，叫Nested-Loops Join（嵌套循环联接）。我们在真正使用MySQL的时候表动不动就是几百上千万数据，如果都按照Nested-Loops Join算法，一次Join查询的代价也太大了。所以下面就来看看MySQL支持的Join算法都有哪些？

## 1.2.联接算法介绍

联接算法是MySQL数据库用于处理联接的物理策略。目前MySQL数据库仅支持Nested-Loops Join算法。而MySQL的分支版本MariaDB除了支持Nested-Loops Join算法外，还支持Classic Hash Join算法。当联接的表上有索引时，**Nested-Loops Join是非常高效的算法。根据B+树的特性，其联接的时间复杂度为O\(N\)，若没有索引，则可视为最坏的情况，时间复杂度为O\(N²\)。MySQL数据库根据不同的使用场合，支持两种Nested-Loops Join算法，一种是Simple Nested-Loops Join（NLJ）算法，另一种是Block Nested-Loops Join（BNL）算法。**

在讲述MySQL的Join类型与算法前，看看两张表的Join的过程：

![](/static/image/201808030249545.jpg)

上图的Fetch阶段是指当内表关联的列是辅助索引时，但是需要访问表中的数据，那么这时就需要再访问主键索引才能得到数据的过程，**不论表的存储引擎是InnoDB存储引擎还是MyISAM，这都是无法避免的，只是MyISAM的回表速度要快点，因为其辅助索引存放的就是指向记录的指针，而InnoDB存储引擎是索引组织表，需要再次通过索引查找才能定位数据。**

Fetch阶段也不是必须存在的，如果是聚集索引联接，那么直接就能得到数据，无需回表，也就没有Fetch这个阶段。另外，上述给出了两张表之间的Join过程，多张表的Join就是继续上述这个过程。

接着计算两张表Join的成本，这里有下列几种概念：

```
外表的扫描次数，记为O。通常外表的扫描次数都是1，即Join时扫描一次外表（驱动表）的数据即可

内表的扫描次数，记为I。根据不同Join算法，内表（被驱动表）的扫描次数不同

读取表的记录数，记为R。根据不同Join算法，读取记录的数量可能不同

Join的比较次数，记为M。根据不同Join算法，比较次数不同

回表的读取记录的数，记为F。若Join的是辅助索引，可能需要回表取得最终的数据
```

评判一个Join算法是否优劣，就是查看上述这些操作的开销是否比较小。当然，这还要考虑I/O的访问方式，顺序还是随机，总之Join的调优也是门艺术，并非想象的那么简单。

### Simple Nested-Loops Join（SNLJ，简单嵌套循环联接）

Simple Nested-Loops Join算法相当简单、直接。即外表（驱动表）中的每一条记录与内表（被驱动表）中的记录进行比较判断。对于两表连接来说，驱动表只会被访问一遍，但被驱动表却要被访问到好多遍，具体访问几遍取决于对驱动表执行单表查询后的结果集中的记录条数。对于内连接来说，选取哪个表为驱动表都没关系，而外连接的驱动表是固定的，也就是说左（外）连接的驱动表就是左边的那个表，右（外）连接的驱动表就是右边的那个表。

用伪代码表示一下这个过程就是这样：

```
For each row r in R do                         -- 扫描R表（驱动表）
    For each row s in S do                     -- 扫描S表（被驱动表）
        If r and s satisfy the join condition  -- 如果r和s满足join条件
            Then output the tuple <r, s>       -- 返回结果集
```

下图能更好地显示整个SNLJ的过程：

![](/static/image/2018080111305498.jpg)

其中R表为外部表（Outer Table），S表为内部表（Inner Table）。这是一个最简单的算法，这个算法的开销其实非常大。假设在两张表R和S上进行联接的列都不含有索引，外表的记录数为RN，内表的记录数位SN。根据上一节对于Join算法的评判标准来看，SNLJ的开销如下表所示：

| 开销统计 | SNLJ |
| :--- | :--- |
| 外表扫描次数（O） | 1 |
| 内表扫描次数（I） | RN |
| 读取记录数（R） | RN + SN\*RN |
| Join比较次数（M） | SN\*RN |
| 回表读取记录次数（F） | 0 |

可以看到读取记录数的成本和比较次数的成本都是SN\*RN，也就是笛卡儿积。假设外表内表都是1万条记录，那么其读取的记录数量和Join的比较次数都需要上亿。实际上数据库并不会使用到SNLJ算法。

### Index Nested-Loops Join（INLJ，基于索引的嵌套循环联接）

SNLJ算法虽然简单明了，但是也是相当的粗暴，需要多次访问内表（每一次都是全表扫描）。因此，在Join的优化时候，通常都会建议在内表建立索引，以此降低Nested-Loop Join算法的开销，减少内表扫描次数，MySQL数据库中使用较多的就是这种算法，以下称为INLJ。来看这种算法的伪代码：

```
For each row r in R do                     -- 扫描R表
    lookup s in S index                    -- 查询S表的索引（固定3~4次IO，B+树高度）
        If find s == r                     -- 如果r匹配了索引s
            Then output the tuple <r, s>   -- 返回结果集
```

由于内表上有索引，所以比较的时候不再需要一条条记录进行比较，而可以通过索引来减少比较，从而加速查询。整个过程如下图所示：

![](/static/image/2018080111472193.jpg)

可以看到外表中的每条记录通过内表的索引进行访问，就是读取外部表一行数据，然后去内部表索引进行二分查找匹配；而一般B+树的高度为3~4层，也就是说匹配一次的io消耗也就3~4次，因此索引查询的成本是比较固定的，故优化器都倾向于使用记录数少的表作为外表（这里是否又会存在潜在的问题呢？）。故INLJ的算法成本如下表所示：

| 开销统计 | SNLJ | INLJ |
| :--- | :--- | :--- |
| 外表扫描次数（O） | 1 | 1 |
| 内表扫描次数（I） | R | 0 |
| 读取记录数（R） | RN + SN\*RN | RN + Smatch |
| Join比较次数（M） | SN\*RN | RN \* IndexHeight |
| 回表读取记录次数（F） | 0 | Smatch \(if possible\) |

上表Smatch表示通过索引找到匹配的记录数量。同时可以发现，通过索引可以大幅降低内表的Join的比较次数，每次比较1条外表的记录，其实就是一次indexlookup（索引查找），而每次index lookup的成本就是树的高度，即IndexHeight。

INLJ的算法并不复杂，也算简单易懂。但是效率是否能达到用户的预期呢？其实如果是通过表的主键索引进行Join，即使是大数据量的情况下，INLJ的效率亦是相当不错的。因为索引查找的开销非常小，并且访问模式也是顺序的（假设大多数聚集索引的访问都是比较顺序的）。

大部分人诟病MySQL的INLJ慢，主要是因为在进行Join的时候可能用到的索引并不是主键的聚集索引，而是辅助索引，这时INLJ的过程又需要多一步Fetch的过程，而且这个过程开销会相当的大：

![](/static/image/2018080111284989.jpg)

由于访问的是辅助索引，如果查询需要访问聚集索引上的列，那么必要需要进行回表取数据，看似每条记录只是多了一次回表操作，但这才是INLJ算法最大的弊端。首先，辅助索引的index lookup是比较随机I/O访问操作。其次，根据index lookup再进行回表又是一个随机的I/O操作。所以说，INLJ最大的弊端是其可能需要大量的离散操作，这在SSD出现之前是最大的瓶颈。而即使SSD的出现大幅提升了随机的访问性能，但是对比顺序I/O，其还是慢了很多，依然不在一个数量级上。

另外，在INNER JOIN中，两张联接表的顺序是可以变换的，即R INNER JOIN S ON Condition P等效于S INNER JOIN R ON Condition P。根据前面描述的Simple Nested-Loops Join算法，优化器在一般情况下总是选择将联接列含有索引的表作为内部表。如果两张表R和S在联接列上都有索引，并且索引的高度相同，那么优化器会选择记录数少的表作为外部表，这是因为内部表的扫描次数总是索引的高度，与记录的数量无关。所以，联接列只要有一个字段有索引即可，但最好是数据集多的表有索引；但是，但有WHERE条件的时候又另当别论了。

然后我们给上面的 t1.m1 和 t2.m2 分别添加主键，看一下下面这个内联接的执行计划：

```
mysql> EXPLAIN SELECT * FROM t1 INNER JOIN t2 on t1.m1 = t2.m2;
+----+-------------+-------+------------+--------+---------------+---------+---------+-----------------+------+----------+-------+
| id | select_type | table | partitions | type   | possible_keys | key     | key_len | ref             | rows | filtered | Extra |
+----+-------------+-------+------------+--------+---------------+---------+---------+-----------------+------+----------+-------+
|  1 | SIMPLE      | t1    | NULL       | ALL    | PRIMARY       | NULL    | NULL    | NULL            |    3 |   100.00 | NULL  |
|  1 | SIMPLE      | t2    | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | employees.t1.m1 |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+--------+---------------+---------+---------+-----------------+------+----------+-------+
2 rows in set, 1 warning (0.00 sec)
```

可以看到执行计划是将 t1 表作为驱动表，将 t2 表作为被驱动表，因为对 t2.m2 列的条件是等值查找，比如 t2.m2=2、t2.m2=3 等，所以MySQL把在联接查询中对被驱动表使用主键值或者唯一二级索引列的值进行等值查找的查询执行方式称之为eq\_ref。

```
Tips：如果被驱动表使用了非唯一二级索引列的值进行等值查询，则查询方式为 ref。另外，如果被驱动表使用了主键或者唯一二级索引列的值进行等值查找，但主键或唯一二级索引如果有多个列的话，则查询类型也会变成 ref。

有时候联接查询的查询列表和过滤条件中可能只涉及被驱动表的部分列，而这些列都是某个索引的一部分，这种情况下即使不能使用eq_ref、ref、ref_or_null或者range这些访问方法执行对被驱动表的查询的话，也可以使用索引扫描，也就是index的访问方法来查询被驱动表。所以我们建议在真实工作中最好不要使用*作为查询列表，最好把真实用到的列作为查询列表。
```

这里为什么将 t1 作为驱动表？因为表 t1 中的记录少于表 t2，这样联接需要匹配的次数就少了，所以SQL优化器选择表 t1 作为驱动表。

若我们执行的SQL带有WHERE条件时呢？看看不一样的执行计划。如果条件为表 t1 的主键，执行计划如下：

```
mysql> EXPLAIN SELECT * FROM t1 INNER JOIN t2 on t1.m1 = t2.m2 WHERE t1.m1 = 2;
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | t1    | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
|  1 | SIMPLE      | t2    | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
2 rows in set, 1 warning (0.00 sec)
```

可以看到执行计划算是极优，同时 t1 表还是驱动表，因为经过WHERE条件过滤后的数据只有一条（我们知道在单表中使用主键值或者唯一二级索引列的值进行等值查找的方式称之为const，所以我们可以看到 t1 的type为const；如果这里条件为 t1.m1 &gt; 1，那么自然 type 就为 range），同时 t2.m2 也是主键，自然只有一条数据，type也为const。

如果WHERE条件是一个没有索引的字段呢？执行计划如下：

```
mysql> EXPLAIN SELECT * FROM t1 INNER JOIN t2 on t1.m1 = t2.m2 WHERE t1.n1='a';
+----+-------------+-------+------------+--------+---------------+---------+---------+-----------------+------+----------+-------------+
| id | select_type | table | partitions | type   | possible_keys | key     | key_len | ref             | rows | filtered | Extra       |
+----+-------------+-------+------------+--------+---------------+---------+---------+-----------------+------+----------+-------------+
|  1 | SIMPLE      | t1    | NULL       | ALL    | PRIMARY       | NULL    | NULL    | NULL            |    3 |    33.33 | Using where |
|  1 | SIMPLE      | t2    | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | employees.t1.m1 |    1 |   100.00 | NULL        |
+----+-------------+-------+------------+--------+---------------+---------+---------+-----------------+------+----------+-------------+
2 rows in set, 1 warning (0.00 sec)
```

从执行计划上看跟不加WHERE条件几乎差不多，但是可以看到filtered为33%了，而不是100%，说明需要返回的数据量变少了。另外Extra字段中标识使用了WHERE条件过滤。

如果WHERE条件是一个有索引的字段呢（比如给 t2.n2 添加一个非唯一二级索引）？这里就不得不提MySQL一个非常重要的特性了，pushed-down conditions（条件下推）优化。就是把索引条件下推到存储引擎层进行数据的过滤并返回过滤后的数据。那么此时的执行计划就如下：

```
mysql> EXPLAIN SELECT * FROM t1 INNER JOIN t2 on t1.m1 = t2.m2 WHERE t2.n2='a';
+----+-------------+-------+------------+--------+----------------+---------+---------+-----------------+------+----------+-------------+
| id | select_type | table | partitions | type   | possible_keys  | key     | key_len | ref             | rows | filtered | Extra       |
+----+-------------+-------+------------+--------+----------------+---------+---------+-----------------+------+----------+-------------+
|  1 | SIMPLE      | t2    | NULL       | ref    | PRIMARY,idx_n2 | idx_n2  | 2       | const           |    1 |   100.00 | Using index |
|  1 | SIMPLE      | t1    | NULL       | eq_ref | PRIMARY        | PRIMARY | 4       | employees.t2.m2 |    1 |   100.00 | NULL        |
+----+-------------+-------+------------+--------+----------------+---------+---------+-----------------+------+----------+-------------+
2 rows in set, 1 warning (0.00 sec)
```

可以看到 t2 表成为了驱动表（经过二级索引过滤后数据只有1条，所以这里使用到ref的访问方法）。

如果我们把 t2.n2 换为范围查询呢？看执行计划如下：

```
mysql> EXPLAIN SELECT * FROM t1 INNER JOIN t2 on t1.m1 = t2.m2 WHERE t2.n2>'a';
+----+-------------+-------+------------+--------+----------------+---------+---------+-----------------+------+----------+-------------+
| id | select_type | table | partitions | type   | possible_keys  | key     | key_len | ref             | rows | filtered | Extra       |
+----+-------------+-------+------------+--------+----------------+---------+---------+-----------------+------+----------+-------------+
|  1 | SIMPLE      | t1    | NULL       | ALL    | PRIMARY        | NULL    | NULL    | NULL            |    3 |   100.00 | NULL        |
|  1 | SIMPLE      | t2    | NULL       | eq_ref | PRIMARY,idx_n2 | PRIMARY | 4       | employees.t1.m1 |    1 |   100.00 | Using where |
+----+-------------+-------+------------+--------+----------------+---------+---------+-----------------+------+----------+-------------+
2 rows in set, 1 warning (0.00 sec)
```

可以看到虽然WHERE条件有索引，但由于 t2.n2&gt;’a’ 过滤后的数据还是比 t1 表多，所以优化器就选择了 t1 表作为驱动表。而此时 t2 表的查询条件类似如下：

```
SELECT * FROM t2 WHERE t2.m2 = 1 AND t2.n2 > 'a';
```

由于 t2.m2 是主键，t2.n2 有二级索引，优化器平衡了一下，可能觉得 t2.n2 过滤后的数据占全表比例太大，回表的成本比直接访问主键成本要高，所以就直接使用了主键。如果说 t2.n2 过滤后的数据占全表数据比例较小，是有可能会选择 idx\_n2 索引。

最后，我们使用 t1.n1 与 t2.n2 作为条件，看一下执行计划如下：

```
mysql> EXPLAIN SELECT * FROM t1 INNER JOIN t2 on t1.n1 = t2.n2;
+----+-------------+-------+------------+------+---------------+--------+---------+-----------------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key    | key_len | ref             | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+--------+---------+-----------------+------+----------+-------------+
|  1 | SIMPLE      | t1    | NULL       | ALL  | NULL          | NULL   | NULL    | NULL            |    3 |   100.00 | Using where |
|  1 | SIMPLE      | t2    | NULL       | ref  | idx_n2        | idx_n2 | 1       | employees.t1.n1 |    1 |   100.00 | Using index |
+----+-------------+-------+------------+------+---------------+--------+---------+-----------------+------+----------+-------------+
2 rows in set, 1 warning (0.00 sec)
```

一切按照我们预想的结果在工作，就是由于 t2.n2 不是主键或唯一索引，type类型变成了 ref。

```
Tips：虽然在INNER JOIN中可以使用pushed-down conditions的优化方式，但是不能直接在OUTER JOIN中使用该方式，因为有些不满足联接条件的记录会通过外部表行的方式再次添加到结果中，因此需要有条件地使用pushed-down conditions的优化。在优化器内部对于联接查询会设置一个标志来表示是否启用pushed-down conditions的过滤。
```

### Block Nested-Loops Join（BNL，基于块的嵌套循环联接）

**扫描一个表的过程其实是先把这个表从磁盘上加载到内存中，然后从内存中比较匹配条件是否满足。但内存里可能并不能完全存放的下表中所有的记录，所以在扫描表前边记录的时候后边的记录可能还在磁盘上，等扫描到后边记录的时候可能内存不足，所以需要把前边的记录从内存中释放掉。**我们前边又说过，采用Simple Nested-Loop Join算法的两表连接过程中，被驱动表可是要被访问好多次的，如果这个被驱动表中的数据特别多而且不能使用索引进行访问，那就相当于要从磁盘上读好几次这个表，这个I/O代价就非常大了，所以我们得想办法：尽量减少访问被驱动表的次数。

当被驱动表中的数据非常多时，每次访问被驱动表，被驱动表的记录会被加载到内存中，在内存中的每一条记录只会和驱动表结果集的一条记录做匹配，之后就会被从内存中清除掉。然后再从驱动表结果集中拿出另一条记录，再一次把被驱动表的记录加载到内存中一遍，周而复始，驱动表结果集中有多少条记录，就得把被驱动表从磁盘上加载到内存中多少次。所以我们可不可以在把被驱动表的记录加载到内存的时候，一次性和多条驱动表中的记录做匹配，这样就可以大大减少重复从磁盘上加载被驱动表的代价了。这也就是Block Nested-Loop Join算法的思想。

也就是说在有索引的情况下，MySQL会尝试去使用Index Nested-Loop Join算法，在有些情况下，可能Join的列就是没有索引，那么这时MySQL的选择绝对不会是最先介绍的Simple Nested-Loop Join算法，因为那个算法太粗暴，不忍直视。数据量大些的复杂SQL估计几年都可能跑不出结果。而Block Nested-Loop Join算法较Simple Nested-Loop Join的改进就在于可以减少内表的扫描次数，甚至可以和Hash Join算法一样，仅需扫描内表一次。其使用Join Buffer（联接缓冲）来减少内部循环读取表的次数。

```
For each tuple r in R do                             -- 扫描外表R
    store used columns as p from R in Join Buffer    -- 将部分或者全部R的记录保存到Join Buffer中，记为p
    For each tuple s in S do                         -- 扫描内表S
        If p and s satisfy the join condition        -- p与s满足join条件
            Then output the tuple                    -- 返回为结果集
```

可以看到相比Simple Nested-Loop Join算法，Block Nested-LoopJoin算法仅多了一个所谓的Join Buffer，为什么这样就能减少内表的扫描次数呢？下图相比更好地解释了Block Nested-Loop Join算法的运行过程：

![](/static/image/2018080112323783.jpg)

MySQL数据库使用Join Buffer的原则如下：

* 系统变量Join\_buffer\_size决定了Join Buffer的大小。

* Join Buffer可被用于联接是ALL、index、和range的类型。

* 每次联接使用一个Join Buffer，因此多表的联接可以使用多个Join Buffer。

* Join Buffer在联接发生之前进行分配，在SQL语句执行完后进行释放。

* Join Buffer只存储要进行查询操作的相关列数据，而不是整行的记录。

**Join\_buffer\_size变量**

所以，Join Buffer并不是那么好用的。首先变量join\_buffer\_size用来控制Join Buffer的大小，调大后可以避免多次的内表扫描，从而提高性能。也就是说，当MySQL的Join有使用到Block Nested-Loop Join，那么调大变量join\_buffer\_size才是有意义的。而前面的Index Nested-Loop Join如果仅使用索引进行Join，那么调大这个变量则毫无意义。

变量join\_buffer\_size的默认值是256K，显然对于稍复杂的SQL是不够用的。好在这个是会话级别的变量，可以在执行前进行扩展。建议在会话级别进行设置，而不是全局设置，因为很难给一个通用值去衡量。另外，这个内存是会话级别分配的，如果设置不好容易导致因无法分配内存而导致的宕机问题。

**Join Buffer缓存对象**

另外，Join Buffer缓存的对象是什么，这个问题相当关键和重要。然在MySQL的官方手册中是这样记录的：Only columns of interest to the join are stored in the join buffer, not whole rows.

可以发现Join Buffer不是缓存外表的整行记录，而是缓存“columns of interest”，具体指所有参与查询的列都会保存到Join Buffer，而不是只有Join的列。比如下面的SQL语句，假设没有索引，需要使用到Join Buffer进行链接：

```
SELECT a.col3
FROM a,
     b
WHERE a.col1 = b.col2
  AND a.col2 > ….
  AND b.col2 = …
```

假设上述SQL语句的外表是a，内表是b，那么存放在Join Buffer中的列是所有参与查询的列，在这里就是（a.col1，a.col2，a.col3）。

通过上面的介绍，我们现在可以得到内表的扫描次数为：

```
Scaninner_table = (RN * used_column_size) / join_buffer_size + 1
```

对于有经验的DBA就可以预估需要分配的Join Buffer大小，然后尽量使得内表的扫描次数尽可能的少，最优的情况是只扫描内表一次。

**Join Buffer的分配**  
需要牢记的是，Join Buffer是在Join之前就进行分配，并且每次Join就需要分配一次Join Buffer，所以假设有N张表参与Join，每张表之间通过Block Nested-Loop Join，那么总共需要分配N-1个Join Buffer，这个内存容量是需要DBA进行考量的。

在MySQL 5.6（包括MariaDB 5.3）中，优化了Join Buffer在多张表之间联接的内存使用效率。MySQL 5.6将Join Buffer分为Regular join buffer和Incremental join buffer。假设B1是表t1和t2联接使用的Join Buffer，B2是t1和t2联接产生的结果和表t3进行联接使用的join buffer，那么：

* 如果B2是Regular join buffer，那么B2就会包含B1的Join Buffer中r1相关的列，以及表t2中相关的列。

* 如果B2是Incremental join buffer，那么B2包含表t2中的数据及一个指针，该指针指向B1中r1相对应的数据。

因此，对于第一次联接的表，使用的都是Regular join buffer，之后再联接，则使用Incremental join buffer。又因为Incremental join buffer只包含指向之前Join Buffer中数据的指针，所以Join Buffer的内存使用效率得到了大幅的提高。

此外，对于NULL类型的列，其实不需要存放在Join Buffer中，而对于VARCHAR类型的列，也是仅需最小的内存即可，而不是以CHAR类型在Join Buffer中保存。最后，在MySQL 5.5版本中，Join Buffer只能在INNER JOIN中使用，在OUTER JOIN中则不能使用，即Block Nested Loop算法不支持OUTER JOIN。从MySQL 5.6及MariaDB 5.3开始，Join Buffer的使用得到了进一步扩展，在OUTER JOIN中使join buffer得到支持。

**Block Nested-Loop Join开销**

Block Nested-Loop Join极大的避免了内表的扫描次数，如果Join Buffer可以缓存外表的数据，那么内表的扫描仅需一次，这和Hash Join非常类似。但是Block Nested-Loop Join依然没有解决的是Join比较的次数，其仍然通过Join判断式进行比较。综上所述，到目前为止各Join算法的成本比较如下所示：

| 开销统计 | SNLJ | INLJ | BNL |
| :--- | :--- | :--- | :--- |
| 外表扫描次数（O） | 1 | 1 | 1 |
| 内表扫描次数（I） | R | 0 | RN\*used\_column\_size/join\_buffer\_size + 1 |
| 读取记录数（R） | RN + SN\*RN | RN + Smatch | RN + S\*I |
| Join比较次数（M） | SN\*RN | RN \* IndexHeight | SN\*RN |
| 回表读取记录次数（F） | 0 | Smatch \(if possible\) | 0 |

这个算法很好测试，我们可以随便构建两张没有索引的字段进行联接，然后查看一下执行计划。下面是我在MySQL 5.7版本上的执行计划。

```
mysql> EXPLAIN SELECT * FROM t1 INNER JOIN t2 on t1.m1 = t2.m2 WHERE t2.n2>'c';
+----+-------------+-------+------------+-------+----------------+--------+---------+------+------+----------+----------------------------------------------------+
| id | select_type | table | partitions | type  | possible_keys  | key    | key_len | ref  | rows | filtered | Extra                                              |
+----+-------------+-------+------------+-------+----------------+--------+---------+------+------+----------+----------------------------------------------------+
|  1 | SIMPLE      | t2    | NULL       | range | PRIMARY,idx_n2 | idx_n2 | 2       | NULL |    3 |   100.00 | Using where; Using index                           |
|  1 | SIMPLE      | t1    | NULL       | ALL   | PRIMARY        | NULL   | NULL    | NULL |    3 |    33.33 | Using where; Using join buffer (Block Nested Loop) |
+----+-------------+-------+------------+-------+----------------+--------+---------+------+------+----------+----------------------------------------------------+
2 rows in set, 1 warning (0.00 sec)
```

可以看到，SQL 执行计划的 Extra 列中提示 Using join buffer \(Block Nested Loop\)，很明显使用了BNL算法。

另外，可以看出这条 SQL 先根据索引进行了条件过滤，然后拿过滤后的结果集作为驱动表，也是为了减少被驱动表扫描次数。如果 t2.n2 没有索引呢？使用 BNL 算法来 join 的话，这个语句的执行流程是这样的，假设表 t1 是驱动表，表 t2 是被驱动表：

1. 把表 t1 的所有字段取出来，存入 join\_buffer 中。

2. 扫描表 t2，取出每一行数据跟 join\_buffer 中的数据进行对比；如果不满足 t1.m1=t2.m2，则跳过； 如果满足 t1.m1=t2.m2，再判断其他条件，也就是是否满足 t2.n2&gt;’c’ 的条件，如果是，就作为结果集的一部分返回，否则跳过。

对于表 t2 的每一行，判断 join 是否满足的时候，都需要遍历 join\_buffer 中的所有行。因此判断等值条件的次数是 t1表行数\*t2表行数，数据量稍微大点时，这个判断的次数都是上亿次。如果不想在表 t2 的字段 n2 上创建索引，又想减少比较次数。那么，有没有两全其美的办法呢？这时候，我们可以考虑使用临时表。使用临时表的大致思路是：

1. 把表 t2 中满足条件的数据放在临时表 tmp\_t 中；

2. 为了让 join 使用 BKA 算法，给临时表 tmp\_t 的字段 n2 加上索引；

3. 让表 t1 和 tmp\_t 做 join 操作。

**Block Nested-Loop Join影响**

在使用 Block Nested-Loop Join\(BNL\) 算法时，可能会对被驱动表做多次扫描。如果这个被驱动表是一个大的冷数据表，除了会导致 IO 压力大以外，还会对 buffer poll 产生严重的影响。

如果了解 InnoDB 的 LRU 算法就会知道，由于 InnoDB 对 Bufffer Pool 的 LRU 算法做了优化，即：第一次从磁盘读入内存的数据页，会先放在 old 区域。如果 1 秒之后这个数据页不再被访问了，就不会被移动到 LRU 链表头部，这样对 Buffer Pool 的命中率影响就不大。

但是，如果一个使用 BNL 算法的 join 语句，多次扫描一个冷表，而且这个语句执行时间超过 1 秒，就会在再次扫描冷表的时候，把冷表的数据页移到 LRU 链表头部。这种情况对应的，是冷表的数据量小于整个 Buffer Pool 的 3/8，能够完全放入 old 区域的情况。如果这个冷表很大，就会出现另外一种情况：业务正常访问的数据页，没有机会进入 young 区域。

由于优化机制的存在，一个正常访问的数据页，要进入 young 区域，需要隔 1 秒后再次被访问到。但是，由于我们的 join 语句在循环读磁盘和淘汰内存页，进入 old 区域的数据页，很可能在 1 秒之内就被淘汰了。这样，就会导致这个 MySQL 实例的 Buffer Pool 在这段时间内，young 区域的数据页没有被合理地淘汰。

也就是说，这两种情况都会影响 Buffer Pool 的正常运作。 大表 join 操作虽然对 IO 有影响，但是在语句执行结束后，对 IO 的影响也就结束了。但是，对 Buffer Pool 的影响就是持续性的，需要依靠后续的查询请求慢慢恢复内存命中率。

为了减少这种影响，你可以考虑增大 join\_buffer\_size 的值，减少对被驱动表的扫描次数。

也就是说，BNL 算法对系统的影响主要包括三个方面： 可能会多次扫描被驱动表，占用磁盘 IO 资源； 判断 join 条件需要执行 M\*N 次对比（M、N 分别是两张表的行数），如果是大表就会占用非常多的 CPU 资源； 可能会导致 Buffer Pool 的热数据被淘汰，影响内存命中率。

**Batched Key Access Join（BKA，批量键访问联接）**  
Index Nested-Loop Join虽好，但是通过辅助索引进行联接后需要回表，这里需要大量的随机I/O操作。若能优化随机I/O，那么就能极大的提升Join的性能。为此，MySQL 5.6（MariaDB 5.3）开始支持Batched Key Access Join算法（简称BKA），该算法通过常见的空间换时间，随机I/O转顺序I/O，以此来极大的提升Join的性能。

在说明Batched Key Access Join前，首先介绍下MySQL 5.6的新特性mrr——multi range read。因为这个特性也是BKA的重要支柱。MRR优化的目的就是为了减少磁盘的随机访问，InnoDB由于索引组织表的特性，如果你的查询是使用辅助索引，并且有用到表中非索引列（投影非索引字段，及条件有非索引字段），因此需要回表读取数据做后续处理，过于随机的回表会伴随着大量的随机I/O。这个过程如下图所示：

![](/static/image/2018080113315135.jpg)

而mrr的优化在于，并不是每次通过辅助索引读取到数据就回表去取记录，范围扫描（range access）中MySQL将扫描到的数据存入由 read\_rnd\_buffer\_size 变量定义的内存大小中，默认256K。然后对其按照Primary Key（RowID）排序，然后使用排序好的数据进行顺序回表，因为我们知道InnoDB中叶子节点数据是按照PRIMARY KEY（ROWID）进行顺序排列的，所以我们可以认为，如果按照主键的递增顺序查询的话，对磁盘的读比较接近顺序读，能够提升读性能。这对于IO-bound类型的SQL查询语句带来性能极大的提升。

MRR 能够提升性能的核心在于，这条查询语句在索引上做的是一个范围查询（也就是说，这是一个多值查询），可以得到足够多的主键id。这样通过排序以后，再去主键索引查数据，才能体现出“顺序性”的优势。所以MRR优化可用于range，ref，eq\_ref类型的查询，工作方式如下图：

![](/static/image/2016120912190540.png)

要开启mrr还有一个比较重的参数是在变量optimizer\_switch中的mrr和mrr\_cost\_based选项。mrr选项默认为on，mrr\_cost\_based选项默认为off。mrr\_cost\_based选项表示通过基于成本的算法来确定是否需要开启mrr特性。然而，在MySQL当前版本中，基于成本的算法过于保守，导致大部分情况下优化器都不会选择mrr特性。为了确保优化器使用mrr特性，请执行下面的SQL语句：

```
set optimizer_switch='mrr=on,mrr_cost_based=off';
```

但如果强制开启MRR，那在某些SQL语句下，性能可能会变差；因为MRR需要排序，假如排序的时间超过直接扫描的时间，那性能就会降低。optimizer\_switch可以是全局的，也可以是会话级的。

当然，除了调整参数外，数据库也提供了语句级别的开启或关闭MRR，使用方法如下：

```
mysql> explain select /*+ MRR(employees)*/ * from employees where birth_date >= '1996-01-01'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: employees
   partitions: NULL
         type: range
possible_keys: idx_birth_date
          key: idx_birth_date
      key_len: 3
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: Using index condition; Using MRR
1 row in set, 1 warning (0.00 sec)
```

理解了 MRR 性能提升的原理，我们就能理解 MySQL 在 5.6 版本后开始引入的 Batched Key Acess\(BKA\) 算法了。这个 BKA 算法，其实就是对 INLJ 算法的优化。

我们知道 INLJ 算法执行的逻辑是：从驱动表一行行地取出 join 条件值，再到被驱动表去做 join。也就是说，对于被驱动表来说，每次都是匹配一个值。这时，MRR 的优势就用不上了。那怎么才能一次性地多传些值给被驱动表呢？方法就是，从驱动表里一次性地多拿些行出来，一起传给被驱动表。既然如此，我们就把驱动表的数据取出来一部分，先放到一个临时内存。这个临时内存不是别人，就是 join\_buffer。

我们知道 join\_buffer 在 BNL 算法里的作用，是暂存驱动表的数据。但是在 NLJ 算法里并没有用。那么，我们刚好就可以复用 join\_buffer 到 BKA 算法中。NLJ 算法优化后的 BKA 算法的流程，整个过程如下所示：

![](/static/image/2018080202505519.jpg)

对于多表join语句，当MySQL使用索引访问第二个join表的时候，使用一个join buffer来收集第一个操作对象生成的相关列值。BKA构建好key后，批量传给引擎层做索引查找。key是通过MRR接口提交给引擎的，这样，MRR使得查询更有效率。

如果外部表扫描的是主键，那么表中的记录访问都是比较有序的，但是如果联接的列是非主键索引，那么对于表中记录的访问可能就是非常离散的。因此对于非主键索引的联接，Batched Key Access Join算法将能极大提高SQL的执行效率。BKA算法支持内连接，外连接和半连接操作，包括嵌套外连接。

Batched Key Access Join算法的工作步骤如下：

1. 将外部表中相关的列放入Join Buffer中。

2. 批量的将Key（索引键值）发送到Multi-Range Read（MRR）接口。

3. Multi-Range Read（MRR）通过收到的Key，根据其对应的ROWID进行排序，然后再进行数据的读取操作。

4. 返回结果集给客户端。

Batched Key Access Join算法的本质上来说还是Simple Nested-Loops Join算法，其发生的条件为内部表上有索引，并且该索引为非主键，并且联接需要访问内部表主键上的索引。这时Batched Key Access Join算法会调用Multi-Range Read（MRR）接口，批量的进行索引键的匹配和主键索引上获取数据的操作，以此来提高联接的执行效率，因为读取数据是以顺序磁盘IO而不是随机磁盘IO进行的。

在MySQL 5.6中默认关闭BKA（MySQL 5.7默认打开），必须将optimizer\_switch系统变量的batched\_key\_access标志设置为on。BKA使用MRR，因此mrr标志也必须打开。目前，MRR的成本估算过于悲观。因此，mrr\_cost\_based也必须关闭才能使用BKA。以下设置启用BKA：

```
SET optimizer_switch='mrr=on,mrr_cost_based=off,batched_key_access=on';
```

因为BKA算法的本质是通过MRR接口将非主键索引对于记录的访问，转化为根据ROWID排序的较为有序的记录获取，所以要想通过BKA算法来提高性能，不但需要确保联接的列参与match的操作（联接的列可以是唯一索引或者普通索引，但不能是主键），还要有对非主键列的search操作。例如下列SQL语句：

```
mysql> explain select a.gender, b.dept_no from employees a, dept_emp b where a.birth_date=b.from_date;
+----+-------------+-------+------------+------+----------------+----------------+---------+-----------------------+--------+----------+----------------------------------------+
| id | select_type | table | partitions | type | possible_keys  | key            | key_len | ref                   | rows   | filtered | Extra                                  |
+----+-------------+-------+------------+------+----------------+----------------+---------+-----------------------+--------+----------+----------------------------------------+
|  1 | SIMPLE      | b     | NULL       | ALL  | NULL           | NULL           | NULL    | NULL                  | 331570 |   100.00 | NULL                                   |
|  1 | SIMPLE      | a     | NULL       | ref  | idx_birth_date | idx_birth_date | 3       | employees.b.from_date |     62 |   100.00 | Using join buffer (Batched Key Access) |
+----+-------------+-------+------------+------+----------------+----------------+---------+-----------------------+--------+----------+----------------------------------------+
2 rows in set, 1 warning (0.00 sec)
```

列a.gender是表employees的数据，但不是通过搜索idx\_birth\_date索引就能得到数据，还需要回表访问主键来获取数据。因此这时可以使用BKA算法。但是如果联接不涉及针对主键进一步获取数据，内部表只参与联接判断，那么就不会启用BKA算法，因为没有必要去调用MRR接口。比如search的主键（a.emp\_no），那么肯定就不需要BKA算法了，直接覆盖索引就可以返回数据了（二级索引有主键值）。

```
mysql> explain select a.emp_no, b.dept_no from employees a, dept_emp b where a.birth_date=b.from_date;
+----+-------------+-------+------------+------+----------------+----------------+---------+-----------------------+--------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys  | key            | key_len | ref                   | rows   | filtered | Extra       |
+----+-------------+-------+------------+------+----------------+----------------+---------+-----------------------+--------+----------+-------------+
|  1 | SIMPLE      | b     | NULL       | ALL  | NULL           | NULL           | NULL    | NULL                  | 331570 |   100.00 | NULL        |
|  1 | SIMPLE      | a     | NULL       | ref  | idx_birth_date | idx_birth_date | 3       | employees.b.from_date |     62 |   100.00 | Using index |
+----+-------------+-------+------------+------+----------------+----------------+---------+-----------------------+--------+----------+-------------+
2 rows in set, 1 warning (0.00 sec)
```

在EXPLAIN输出中，当Extra值包含Using join buffer（Batched Key Access）且类型值为ref或eq\_ref时，表示使用BKA。

**Classic Hash Join（CHJ）**

MySQL数据库虽然提供了BKA Join来优化传统的JOIN算法，的确在一定程度上可以提升JOIN的速度。但不可否认的是，仍然有许多用户对于Hash Join算法有着强烈的需求。Hash Join不需要任何的索引，通过扫描表就能快速地进行JOIN查询，通过利用磁盘的带宽带最大程度的解决大数据量下的JOIN问题。

MariaDB支持Classic Hash Join算法，该算法不同于Oracle的Grace Hash Join，但是也是通过Hash来进行连接，不需要索引，可充分利用磁盘的带宽。

其实MariaDB的Classic Hash Join和Block Nested Loop Join算法非常类似（Classic Hash Join也称为Block Nested Loop Hash Join），但并不是直接通过进行JOIN的键值进行比较，而是根据Join Buffer中的对象创建哈希表，内表通过哈希算法进行查找，从而在Block Nested Loop Join算法的基础上，又进一步减少了内表的比较次数，从而提升JOIN的查询性能。过程如下图所示：

![](/static/image/2018080206473230.jpg)

Classic Hash Join算法先将外部表中数据放入Join Buffer中，然后根据键值产生一张散列表，这是第一个阶段，称为build阶段。随后读取内部表中的一条记录，对其应用散列函数，将其和散列表中的数据进行比较，这是第二个阶段，称为probe阶段。

如果将Hash查找应用于Simple Nested-Loops Join中，则执行计划的Extra列会显示BNLH。如果将Hash查找应用于Batched Key Access Join中，则执行计划的Extra列会显示BKAH。

同样地，如果Join Buffer能够缓存所有驱动表（外表）的查询列，那么驱动表和内表的扫描次数都将只有1次，并且比较的次数也只是内表记录数（假设哈希算法冲突为0）。反之，需要扫描多次内部表。为了使Classic Hash Join更有效果，应该更好地规划Join Buffer的大小。

要使用Classic Hash Join算法，需要将join\_cache\_level设置为大于等于4的值，并显示地打开优化器的选项，设置过程如下：

# 2.总结

经过上面的学习，我们能发现联接查询成本占大头的就是“驱动表记录数 乘以 单次访问被驱动表的成本”，所以我们的优化重点其实就是下面这两个部分：

* 尽量减少驱动表的记录数

* 对被驱动表的访问成本尽可能降低

这两点对于我们实际书写联接查询语句时十分有用，我们需要尽量在被驱动表的联接列上建立索引（主键或唯一索引最优，其次是非唯一二级索引），这样就可以使用 eq\_ref 或 ref 访问方法来降低访问被驱动表的成本了。

# 3.参考

MySQL联接查询算法（NLJ、BNL、BKA、HashJoin）

[https://blog.csdn.net/weixin\_34326179/article/details/93533355](https://blog.csdn.net/weixin_34326179/article/details/93533355)

