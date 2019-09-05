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



