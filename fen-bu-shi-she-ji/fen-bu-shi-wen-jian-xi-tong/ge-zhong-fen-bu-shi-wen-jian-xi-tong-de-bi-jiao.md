GFS 的 Oracle 翻版，据说性能比 GFS2 好 \(Debian: aptitude install ocfs2-tools, 图形配置工具包 ocfs2console\)

不支持 ACL、flock，只是为了 Oracle各种分布式文件系统的比较

# MooseFS {#moosefs}

支持FUSE，相对比较轻量级，对master服务器有单点依赖，用perl编写，性能相对较差，国内用的人比较多，易用，稳定，对小文件很高效。

```
 + 支持文件元信息
    + mfsmount 很好用
    + 编译依赖少，文档全，默认配置很好
    + mfshdd.cfg 加 * 的条目会被转移到其它 chunk server，以便此 chunk server 安全退出
    + 不要求 chunk server 使用的文件系统格式以及容量一致
    + 开发很活跃
    + 可以以非 root 用户身份运行
    + 可以在线扩容
    + 支持回收站
    + 支持快照
    - master server 存在单点故障
    - master server 很耗内存
```

# MogileFS {#mogilefs}

---

Key-Value型元文件系统，不支持FUSE，应用程序访问它时需要API，主要用在web领域处理海量小图片，效率相比mooseFS高很多，据说对于 Web 2.0 应用存储图片啥的很好。

> 不适合做通用文件系统，适合存储静态只读小文件，比如图片

## GlusterFS

支持FUSE，比mooseFS庞大，感觉广告宣传做的比产品本身好。

* 无单点故障问题

* 支持回收站

* 模块化堆叠式架构

* 对文件系统格式有要求，ext3/ext4/zfs 被正式支持，xfs/jfs 可能可以，reiserfs 经测试可以

* 需要以 root 用户身份运行（用了 trusted xattr，mount 时加 user\_xattr 选项是没用的，官方说法是glusterfsd 需要创建不同属主的文件，所以必需 root 权限\)

* 不能在线扩容\(不 umount 时增加存储节点\)，计划在 3.1 里实现

* 分布存储以文件为单位，条带化分布存储不成熟

# GFS2 {#gfs2}

```
http://sourceware.org/cluster/wiki/DRBD_Cookbook
http://www.smop.co.uk/blog/index.php/2008/02/11/gfs-goodgrief-wheres-the-documentation-file-system/
http://wiki.debian.org/kristian_jerpetjoen
http://longvnit.com/blog/?p=941
http://blog.chinaunix.net/u1/53728/showart_1073271.html (基于红帽RHEL5U2 GFS2+ISCSI+XEN+Cluster 的高可性解决方案)
http://www.yubo.org/blog/?p=27 (iscsi+clvm+gfs2+xen+Cluster)
http://linux.chinaunix.net/bbs/thread-777867-1-1.html
```

并不是 distributed file system, 而是 shared disk cluster file system，需要某种机制在机器

之间共享磁盘，以及加锁机制，因此需要 drbd/iscsi/clvm/ddraid/gnbd 做磁盘共享，以及 dlm 做锁管理\)

* 依赖 Red Hat Cluster Suite \(Debian: aptitude install redhat-cluster-suite， 图形配置工具包 

system-config-cluster, system-config-lvm\)

* 适合不超过约 30 个节点左右的小型集群，规模越大，dlm 的开销越大，默认配置 8 个节点

## OCFS2

GFS 的 Oracle 翻版，据说性能比 GFS2 好 \(Debian: aptitude install ocfs2-tools, 图形配置工具包 ocfs2console\)

不支持 ACL、flock，只是为了 Oracle database 设计

# OpenAFS/Coda {#openafscoda}

---

是很有特色的东西。

```
     + 成熟稳定
    + 开发活跃，支持 Unix/Linux/MacOS X/Windows
    -
 性能不够好
```

# ceph {#ceph}

---

支持FUSE，客户端已经进入了linux-2.6.34内核，也就是说可以像ext3/rasierFS一样，选择ceph为文件系统。彻底的分布式，没有单点依赖，用C编写，性能较好。基于不成熟的btrfs，其本身也非常不成熟

# Lustre {#lustre}

---

Oracle公司的企业级产品，非常庞大，对内核和ext3深度依赖  
复杂，高效，适合大型集群。

```
    * 适合大型集群
    + 很高性能
    + 支持动态扩展
    -
 需要对内核打补丁，深度依赖 
Linux
 内核和 ext3 文件系统
```

## PVFS2

[http://blog.csdn.net/yfw418/archive/2007/07/06/1680930.aspx](http://blog.csdn.net/yfw418/archive/2007/07/06/1680930.aspx)

搭配定制应用会很好，据说曙光的并行文件系统就是基于 PVFS。　　fastDFS：国人在mogileFS的基础上进行改进的key-value型文件系统，同样不支持FUSE，提供比mogileFS更好的性能。

```
\* 高性能

- 没有锁机制，不符合 POSIX 语意，需要应用的配合，不适合做通用文件系统

  \(See pvfs2-guide chaper 5:  PVFS2 User APIs and Semantics\)

- 静态配置，不能动态扩展
```

## Coda

```
\* 从服务器复制文件到本地，文件读写是本地操作因此很高效

\* 文件关闭后发送到服务器

+ 支持离线操作，连线后再同步到服务器上

- 缓存基于文件，不是基于数据块，打开文件时需要等待从服务器缓存到本地完毕

- 并发写有版本冲突问题

- 并发读有极大的延迟，需要等某个 client 关闭文件，比如不适合 tail -f some.log

- 研究项目，不够成熟，使用不广
```

# Hadoop HDFS {#hadoop-hdfs}

---

本地写缓存，够一定大小 \(64 MB\) 时传给服务器  
不适合通用文件系统

# FastDFS {#fastdfs}

---

```
- 只能通过 API 使用，不支持 fuse
```

# NFS {#nfs}

---

老牌网络文件系统，具体不了解，反正NFS最近几年没发展，肯定不能用。

# dCache {#dcache}

---

```
依赖 PostgreSQL
```

# xtreemfs {#xtreemfs}

---

```
* 服务端是 Java 实现的
- 性能不高
```

# CloudStore \(KosmosFS\) {#cloudstore-kosmosfs}

---

```
+ 被 Hadoop 作为分布式文件系统后端之一
- 不支持文件元信息
- kfs_fuse 太慢，不可用
- 编译依赖多，文档落后，脚本简陋
- 开发不活跃
```



