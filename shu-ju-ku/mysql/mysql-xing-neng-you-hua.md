## 1.MySQL 性能优化的那点事儿 {#activity-name}

## 1.1.**MySQL的主要适用场景**

1、Web网站系统

2、日志记录系统

3、数据仓库系统

4、嵌入式系统

## 1.2.**MySQL架构图**

![](/static/image/mysql架构图.webp)

## 1.3.MySQL存储引擎概述

### 1.3.1.MyISAM存储引擎

MyISAM存储引擎的表在数据库中，每一个表都被存放为三个以表名命名的物理文件。首先肯定会有任何存储引擎都不可缺少的存放表结构定义信息的.frm文件，另外还有.MYD和.MYI文件，分别存放了表的数据（.MYD）和索引数据（.MYI）。每个表都有且仅有这样三个文件做为MyISAM存储类型的表的存储，也就是说不管这个表有多少个索引，都是存放在同一个.MYI文件中。

**MyISAM支持以下三种类型的索引：**

**1、B-Tree索引**

B-Tree索引，顾名思义，就是所有的索引节点都按照balancetree的数据结构来存储，所有的索引数据节点都在叶节点。

**2、R-Tree索引**

R-Tree索引的存储方式和b-tree索引有一些区别，主要设计用于为存储空间和多维数据的字段做索引，所以目前的MySQL版本来说，也仅支持geometry类型的字段作索引。

**3、Full-text索引**

Full-text索引就是我们长说的全文索引，他的存储结构也是b-tree。主要是为了解决在我们需要用like查询的低效问题。

### 1.3.2.Innodb 存储引擎

1、支持事务安装

2、数据多版本读取

3、锁定机制的改进

4、实现外键

### 1.3.3.NDBCluster存储引擎

  


NDB存储引擎也叫NDBCluster存储引擎，主要用于MySQLCluster分布式集群环境，Cluster是MySQL从5.0版本才开始提供的新功能。

  


### 1.3.4.Merge存储引擎

  


MERGE存储引擎，在MySQL用户手册中也提到了，也被大家认识为MRG\_MyISAM引擎。Why？因为MERGE存储引擎可以简单的理解为其功能就是实现了对结构相同的MyISAM表，通过一些特殊的包装对外提供一个单一的访问入口，以达到减小应用的复杂度的目的。要创建MERGE表，不仅仅基表的结构要完全一致，包括字段的顺序，基表的索引也必须完全一致。

  


### 1.3.5.Memory存储引擎

  


Memory存储引擎，通过名字就很容易让人知道，他是一个将数据存储在内存中的存储引擎。Memory存储引擎不会将任何数据存放到磁盘上，仅仅存放了一个表结构相关信息的.frm文件在磁盘上面。所以一旦MySQLCrash或者主机Crash之后，Memory的表就只剩下一个结构了。Memory表支持索引，并且同时支持Hash和B－Tree两种格式的索引。由于是存放在内存中，所以Memory都是按照定长的空间来存储数据的，而且不支持BLOB和TEXT类型的字段。Memory存储引擎实现页级锁定。

  


### 1.3.6.BDB存储引擎

  


BDB存储引擎全称为BerkeleyDB存储引擎，和Innodb一样，也不是MySQL自己开发实现的一个存储引擎，而是由SleepycatSoftware所提供，当然，也是开源存储引擎，同样支持事务安全。

  


### 1.3.7.FEDERATED存储引擎

  


FEDERATED存储引擎所实现的功能，和Oracle的DBLINK基本相似，主要用来提供对远程MySQL服务器上面的数据的访问接口。如果我们使用源码编译来安装MySQL，那么必须手工指定启用FEDERATED存储引擎才行，因为MySQL默认是不起用该存储引擎的。

  


### 1.3.8.ARCHIVE存储引擎

  


ARCHIVE存储引擎主要用于通过较小的存储空间来存放过期的很少访问的历史数据。ARCHIVE表不支持索引，通过一个.frm的结构定义文件，一个.ARZ的数据压缩文件还有一个.ARM的meta信息文件。由于其所存放的数据的特殊性，ARCHIVE表不支持删除，修改操

  


作，仅支持插入和查询操作。锁定机制为行级锁定。

  


### 1.3.9.BLACKHOLE存储引擎

  


BLACKHOLE存储引擎是一个非常有意思的存储引擎，功能恰如其名，就是一个“黑洞”。就像我们unix系统下面的“/dev/null”设备一样，不管我们写入任何信息，都是有去无回。

  


### 1.3.10.CSV存储引擎

  


CSV存储引擎实际上操作的就是一个标准的CSV文件，他不支持索引。起主要用途就是大家有些时候可能会需要通过数据库中的数据导出成一份报表文件，而CSV文件是很多软件都支持的一种较为标准的格式，所以我们可以通过先在数据库中建立一张CVS表，然后将生成的报表信息插入到该表，即可得到一份CSV报表文件了。

# 2.参考

MySQL 性能优化的那点事儿：

[https://mp.weixin.qq.com/s?\_\_biz=MzA4Nzg5Nzc5OA==∣=2651667960&idx=1&sn=ea22bdcd724c71d7e1e5669bfdc9b05e&chksm=8bcbfe51bcbc77474e2b378e5bb9a9728bbc106b6e8aa0c3c1bf0e0568ea3dfa5215419ca354&scene=21\#wechat\_redirect](https://mp.weixin.qq.com/s?__biz=MzA4Nzg5Nzc5OA==&mid=2651667960&idx=1&sn=ea22bdcd724c71d7e1e5669bfdc9b05e&chksm=8bcbfe51bcbc77474e2b378e5bb9a9728bbc106b6e8aa0c3c1bf0e0568ea3dfa5215419ca354&scene=21#wechat_redirect)

