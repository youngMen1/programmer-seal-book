# 『浅入浅出』MySQL 和 InnoDB

作为一名开发人员，在日常的工作中会难以避免地接触到数据库，无论是基于文件的 sqlite 还是工程上使用非常广泛的 MySQL、PostgreSQL，但是一直以来也没有对数据库有一个非常清晰并且成体系的认知，所以最近两个月的时间看了几本数据库相关的书籍并且阅读了 MySQL 的官方文档，希望对各位了解数据库的、不了解数据库的有所帮助。

本文中对于数据库的介绍以及研究都是在 MySQL 上进行的，如果涉及到了其他数据库的内容或者实现会在文中单独指出。

## 数据库的定义 {#数据库的定义}

很多开发者在最开始时其实都对数据库有一个比较模糊的认识，觉得数据库就是一堆数据的集合，但是实际却比这复杂的多，数据库领域中有两个词非常容易混淆，也就是_数据库_和_实例_：

* 数据库：物理操作文件系统或其他形式文件类型的集合；
* 实例：MySQL 数据库由后台线程以及一个共享内存区组成；

> 对于数据库和实例的定义都来自于[MySQL 技术内幕：InnoDB 存储引擎](https://book.douban.com/subject/24708143/)一书，想要了解 InnoDB 存储引擎的读者可以阅读这本书籍。

### 数据库和实例 {#数据库和实例}

在 MySQL 中，实例和数据库往往都是一一对应的，而我们也无法直接操作数据库，而是要通过数据库实例来操作数据库文件，可以理解为数据库实例是数据库为上层提供的一个专门用于操作的接口。

Database - Instance.jpg

在 Unix 上，启动一个 MySQL 实例往往会产生两个进程，mysqld 就是真正的数据库服务守护进程，而 mysqld_safe 是一个用于检查和设置 mysqld 启动的控制程序，它负责监控 MySQL 进程的执行，当 mysqld 发生错误时，mysqld_safe 会对其状态进行检查并在合适的条件下重启。

## MySQL 的架构
MySQL 从第一个版本发布到现在已经有了 20 多年的历史，在这么多年的发展和演变中，整个应用的体系结构变得越来越复杂：

Logical-View-of-MySQL-Architecture.jpg

最上层用于连接、线程处理的部分并不是 MySQL 『发明』的，很多服务都有类似的组成部分；第二层中包含了大多数 MySQL 的核心服务，包括了对 SQL 的解析、分析、优化和缓存等功能，存储过程、触发器和视图都是在这里实现的；而第三层就是 MySQL 中真正负责数据的存储和提取的存储引擎，例如：InnoDB、MyISAM 等，文中对存储引擎的介绍都是对 InnoDB 实现的分析。

## 数据的存储
在整个数据库体系结构中，我们可以使用不同的存储引擎来存储数据，而绝大多数存储引擎都以二进制的形式存储数据；这一节会介绍 InnoDB 中对数据是如何存储的。

在 InnoDB 存储引擎中，所有的数据都被逻辑地存放在表空间中，表空间（tablespace）是存储引擎中最高的存储逻辑单位，在表空间的下面又包括段（segment）、区（extent）、页（page）：

Tablespace-segment-extent-page-row.jpg


同一个数据库实例的所有表空间都有相同的页大小；默认情况下，表空间中的页大小都为 16KB，当然也可以通过改变 innodb_page_size 选项对默认大小进行修改，需要注意的是不同的页大小最终也会导致区大小的不同：

Relation Between Page Size - Extent Size.png

从图中可以看出，在 InnoDB 存储引擎中，一个区的大小最小为 1MB，页的数量最少为 64 个。

如何存储表
MySQL 使用 InnoDB 存储表时，会将表的定义和数据索引等信息分开存储，其中前者存储在 .frm 文件中，后者存储在 .ibd 文件中，这一节就会对这两种不同的文件分别进行介绍。

frm-and-ibd-file.jpg

.frm 文件
无论在 MySQL 中选择了哪个存储引擎，所有的 MySQL 表都会在硬盘上创建一个 .frm 文件用来描述表的格式或者说定义；.frm 文件的格式在不同的平台上都是相同的。



```
CREATE TABLE test_frm(
    column1 CHAR(5),
    column2 INTEGER
);

```

当我们使用上面的代码创建表时，会在磁盘上的 datadir 文件夹中生成一个 test_frm.frm 的文件，这个文件中就包含了表结构相关的信息：

frm-file-hex.png

MySQL 官方文档中的 11.1 MySQL .frm File Format 一文对于 .frm 文件格式中的二进制的内容有着非常详细的表述，在这里就不展开介绍了。

## .ibd 文件
InnoDB 中用于存储数据的文件总共有两个部分，一是系统表空间文件，包括 ibdata1、ibdata2 等文件，其中存储了 InnoDB 系统信息和用户数据库表数据和索引，是所有表公用的。

当打开 innodb_file_per_table 选项时，.ibd 文件就是每一个表独有的表空间，文件存储了当前表的数据和相关的索引数据。

## 如何存储记录
与现有的大多数存储引擎一样，InnoDB 使用页作为磁盘管理的最小单位；数据在 InnoDB 存储引擎中都是按行存储的，每个 16KB 大小的页中可以存放 2-200 行的记录。

当 InnoDB 存储数据时，它可以使用不同的行格式进行存储；MySQL 5.7 版本支持以下格式的行存储方式：

Antelope-Barracuda-Row-Format.jpg


Antelope 是 InnoDB 最开始支持的文件格式，它包含两种行格式 Compact 和 Redundant，它最开始并没有名字；Antelope 的名字是在新的文件格式 Barracuda 出现后才起的，Barracuda 的出现引入了两种新的行格式 Compressed 和 Dynamic；InnoDB 对于文件格式都会向前兼容，而官方文档中也对之后会出现的新文件格式预先定义好了名字：Cheetah、Dragon、Elk 等等。

两种行记录格式 Compact 和 Redundant 在磁盘上按照以下方式存储：
COMPACT-And-REDUNDANT-Row-Format.jpg

Compact 和 Redundant 格式最大的不同就是记录格式的第一个部分；在 Compact 中，行记录的第一部分倒序存放了一行数据中列的长度（Length），而 Redundant 中存的是每一列的偏移量（Offset），从总体上上看，Compact 行记录格式相比 Redundant 格式能够减少 20% 的存储空间。

行溢出数据
当 InnoDB 使用 Compact 或者 Redundant 格式存储极长的 VARCHAR 或者 BLOB 这类大对象时，我们并不会直接将所有的内容都存放在数据页节点中，而是将行数据中的前 768 个字节存储在数据页中，后面会通过偏移量指向溢出页。
Row-Overflow.jpg

但是当我们使用新的行记录格式 Compressed 或者 Dynamic 时都只会在行记录中保存 20 个字节的指针，实际的数据都会存放在溢出页面中。

Row-Overflow-in-Barracuda.jpg




