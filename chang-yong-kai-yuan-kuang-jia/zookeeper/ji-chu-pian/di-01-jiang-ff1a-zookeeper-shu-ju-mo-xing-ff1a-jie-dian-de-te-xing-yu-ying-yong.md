# 1.ZooKeeper 数据模型：节点的特性与应用

ZooKeeper 作为一个分布式协调服务，给出了在分布式环境下一致性问题的工业解决方案，目前流行的很多开源框架技术背后都有 ZooKeeper 的身影。那么 ZooKeeper 是如何做到这一点的，在平时开发中我们应该如何使用 ZooKeeper？要想了解这些问题，我们先要对 ZooKeeper 的基础知识进行全面的掌握。

ZooKeeper 基础知识基本分为三大模块：

* 数据模型
* ACL 权限控制
* Watch 监控

其中，数据模型是最重要的，很多 ZooKeeper 中典型的应用场景都是利用这些基础模块实现的。比如我们可以利用数据模型中的临时节点和 Watch 监控机制来实现一个发布订阅的功能。

因此，今天主要通过理论知识结合实际的应用场景来给你介绍数据模型。掌握本课时的知识对于理解 ZooKeeper 内部原理，以及在日常工作中使用好 ZooKeeper 非常重要。

## 数据模型

计算机最根本的作用其实就是处理和存储数据，作为一款分布式一致性框架，ZooKeeper 也是如此。数据模型就是 ZooKeeper 用来存储和处理数据的一种逻辑结构。就像我们用 MySQL 数据库一样，要想处理复杂业务。前提是先学会如何往里边新增数据。ZooKeeper 数据模型最根本的功能就像一个数据库。

现在，数据模型对我们来说还是一个比较抽象的概念，接下来我们开始部署一个开发测试环境，并在上面做一些简单的操作。来看看 ZooKeeper 的数据模型究竟是什么样的：

1.配置文件

```
tickTime=2000
dataDir=/var/lib/zookeeper
clientPort=2181
```

2.服务启动

```
bin/zkServer.sh start
```

3.使用客户端连接服务器

```
bin/zkCli.sh -server 127.0.0.1:2181
```

4.这样单机版的开发环境就已经构建完成了，接下来我们通过 ZooKeeper 提供的 create 命令来创建几个节点，分别是：“/locks”“/servers”“/works”：

```
create /locks
create /servers
create /works
```

最终在 ZooKeeper 服务器上会得到一个具有层级关系的数据结构，如下图所示，这个数据结构就是 ZooKeeper 中的数据模型。

Ciqc1F6yL9OAUUguAAAtBwGI74E989.png

ZooKeeper 中的数据模型是一种树形结构，非常像电脑中的文件系统，有一个根文件夹，下面还有很多子文件夹。ZooKeeper 的数据模型也具有一个固定的根节点（/），我们可以在根节点下创建子节点，并在子节点下继续创建下一级节点。ZooKeeper 树中的每一层级用斜杠（/）分隔开，且只能用绝对路径（如“get /work/task1”）的方式查询 ZooKeeper 节点，而不能使用相对路径。具体的结构你可以看看下面这张图：

CgqCHl6yL9uAbpHYAABF\_GHyGNc950.png

znode 节点类型与特性

知道了 ZooKeeper 的数据模型是一种树形结构，就像在 MySQL 中数据是存在于数据表中，ZooKeeper 中的数据是由多个数据节点最终构成的一个层级的树状结构，和我们在创建 MySOL 数据表时会定义不同类型的数据列字段，ZooKeeper 中的数据节点也分为持久节点、临时节点和有序节点三种类型：

