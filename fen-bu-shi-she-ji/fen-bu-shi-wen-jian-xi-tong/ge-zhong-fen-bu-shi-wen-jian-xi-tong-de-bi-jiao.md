# 各种分布式文件系统的比较

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

————————————————

版权声明：本文为CSDN博主「JeanCheng」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。

原文链接：[https://blog.csdn.net/gatieme/article/details/44982961](https://blog.csdn.net/gatieme/article/details/44982961)

