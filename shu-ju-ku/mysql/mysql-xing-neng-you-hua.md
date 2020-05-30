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

  


# 2.参考

MySQL 性能优化的那点事儿：

[https://mp.weixin.qq.com/s?\_\_biz=MzA4Nzg5Nzc5OA==∣=2651667960&idx=1&sn=ea22bdcd724c71d7e1e5669bfdc9b05e&chksm=8bcbfe51bcbc77474e2b378e5bb9a9728bbc106b6e8aa0c3c1bf0e0568ea3dfa5215419ca354&scene=21\#wechat\_redirect](https://mp.weixin.qq.com/s?__biz=MzA4Nzg5Nzc5OA==&mid=2651667960&idx=1&sn=ea22bdcd724c71d7e1e5669bfdc9b05e&chksm=8bcbfe51bcbc77474e2b378e5bb9a9728bbc106b6e8aa0c3c1bf0e0568ea3dfa5215419ca354&scene=21#wechat_redirect)

