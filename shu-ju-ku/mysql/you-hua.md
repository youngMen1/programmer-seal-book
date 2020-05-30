# [MYSQL性能优化的最佳20+条经验](https://www.cnblogs.com/zhouyusheng/p/8038224.html)

今天，数据库的操作越来越成为整个应用的性能瓶颈了，这点对于Web应用尤其明显。关于数据库的性能，这并不只是DBA才需要担心的事，而这更是我们程序员需要去关注的事情。当我们去设计数据库表结构，对操作数据库时（尤其是查表时的SQL语句），我们都需要注意数据操作的性能。这里，我们不会讲过多的SQL语句的优化，而只是针对MySQL这一Web应用最多的数据库。希望下面的这些优化技巧对你有用。

#### 1. 为查询缓存优化你的查询

大多数的MySQL服务器都开启了查询缓存。这是提高性最有效的方法之一，而且这是被MySQL的数据库引擎处理的。当有很多相同的查询被执行了多次的时候，这些查询结果会被放到一个缓存中，这样，后续的相同的查询就不用操作表而直接访问缓存结果了。

这里最主要的问题是，对于程序员来说，这个事情是很容易被忽略的。因为，我们某些查询语句会让MySQL不使用缓存。请看下面的示例：

```
// 查询缓存不开启
$r = mysql_query("SELECT username FROM user WHERE signup_date >= CURDATE()");

// 开启查询缓存
$today = date("Y-m-d");
$r = mysql_query("SELECT username FROM user WHERE signup_date >= '$today'");
```

上面两条SQL语句的差别就是 CURDATE\(\) ，MySQL的查询缓存对这个函数不起作用。所以，像 NOW\(\) 和 RAND\(\) 或是其它的诸如此类的SQL函数都不会开启查询缓存，因为这些函数的返回是会不定的易变的。所以，你所需要的就是用一个变量来代替MySQL的函数，从而开启缓存。

#### 2. EXPLAIN 你的 SELECT 查询

使用 [EXPLAIN](http://dev.mysql.com/doc/refman/5.0/en/explain.html) 关键字可以让你知道MySQL是如何处理你的SQL语句的。这可以帮你分析你的查询语句或是表结构的性能瓶颈。

EXPLAIN 的查询结果还会告诉你你的索引主键被如何利用的，你的数据表是如何被搜索和排序的……等等，等等。

挑一个你的SELECT语句（推荐挑选那个最复杂的，有多表联接的），把关键字EXPLAIN加到前面。你可以使用phpmyadmin来做这个事。然后，你会看到一张表格。下面的这个示例中，我们忘记加上了group\_id索引，并且有表联接：

![](/static/image/微信截图_20200530154341.png)  
![](/static/image/unoptimized_explain.jpg)

我们可以看到，前一个结果显示搜索了 7883 行，而后一个只是搜索了两个表的 9 和 16 行。查看rows列可以让我们找到潜在的性能问题。

#### 3. 当只要一行数据时使用 LIMIT 1

当你查询表的有些时候，你已经知道结果只会有一条结果，但因为你可能需要去fetch游标，或是你也许会去检查返回的记录数。

在这种情况下，加上 LIMIT 1 可以增加性能。这样一样，MySQL数据库引擎会在找到一条数据后停止搜索，而不是继续往后查少下一条符合记录的数据。

下面的示例，只是为了找一下是否有“中国”的用户，很明显，后面的会比前面的更有效率。（请注意，第一条中是Select \*，第二条是Select 1）

```
// 没有效率的：
$r = mysql_query("SELECT * FROM user WHERE country = 'China'");
if (mysql_num_rows($r) > 0) {
    // ...
}

// 有效率的：
$r = mysql_query("SELECT 1 FROM user WHERE country = 'China' LIMIT 1");
if (mysql_num_rows($r) > 0) {
    // ...
}
```

#### 4. 为搜索字段建索引

索引并不一定就是给主键或是唯一的字段。如果在你的表中，有某个字段你总要会经常用来做搜索，那么，请为其建立索引吧。

![](/static/image/search_index.jpg)

从上图你可以看到那个搜索字串 “last\_name LIKE ‘a%'”，一个是建了索引，一个是没有索引，性能差了4倍左右。

另外，你应该也需要知道什么样的搜索是不能使用正常的索引的。例如，当你需要在一篇大的文章中搜索一个词时，如： “WHERE post\_content LIKE ‘%apple%'”，索引可能是没有意义的。你可能需要使用[MySQL全文索引](http://dev.mysql.com/doc/refman/5.1/en/fulltext-search.html) 或是自己做一个索引（比如说：搜索关键词或是Tag什么的）

#### 5. 在Join表的时候使用相当类型的例，并将其索引

如果你的应用程序有很多 JOIN 查询，你应该确认两个表中Join的字段是被建过索引的。这样，MySQL内部会启动为你优化Join的SQL语句的机制。

而且，这些被用来Join的字段，应该是相同的类型的。例如：如果你要把 DECIMAL 字段和一个 INT 字段Join在一起，MySQL就无法使用它们的索引。对于那些STRING类型，还需要有相同的字符集才行。（两个表的字符集有可能不一样）

```
// 在state中查找company
$r = mysql_query("SELECT company_name FROM users
    LEFT JOIN companies ON (users.state = companies.state)
    WHERE users.id = $user_id");

// 两个 state 字段应该是被建过索引的，而且应该是相当的类型，相同的字符集。
```

#### 6. 千万不要 ORDER BY RAND\(\)

想打乱返回的数据行？随机挑一个数据？真不知道谁发明了这种用法，但很多新手很喜欢这样用。但你确不了解这样做有多么可怕的性能问题。

如果你真的想把返回的数据行打乱了，你有N种方法可以达到这个目的。这样使用只让你的数据库的性能呈指数级的下降。这里的问题是：MySQL会不得不去执行RAND\(\)函数（很耗CPU时间），而且这是为了每一行记录去记行，然后再对其排序。就算是你用了Limit 1也无济于事（因为要排序）

下面的示例是随机挑一条记录

```
// 千万不要这样做：
$r = mysql_query("SELECT username FROM user ORDER BY RAND() LIMIT 1");

// 这要会更好：
$r = mysql_query("SELECT count(*) FROM user");
$d = mysql_fetch_row($r);
$rand = mt_rand(0,$d[0] - 1);

$r = mysql_query("SELECT username FROM user LIMIT $rand, 1");
```

#### 7. 避免 SELECT \*

从数据库里读出越多的数据，那么查询就会变得越慢。并且，如果你的数据库服务器和WEB服务器是两台独立的服务器的话，这还会增加网络传输的负载。

所以，你应该养成一个需要什么就取什么的好的习惯。

```
// 不推荐
$r = mysql_query("SELECT * FROM user WHERE user_id = 1");
$d = mysql_fetch_assoc($r);
echo "Welcome {$d['username']}";
 
// 推荐
$r = mysql_query("SELECT username FROM user WHERE user_id = 1");
$d = mysql_fetch_assoc($r);
echo "Welcome {$d['username']}";
```



