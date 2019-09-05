## [利用zookeeper实现分布式leader节点选举](https://blog.csdn.net/johnson_moon/article/details/78809995)

## 利用zookeeper实现分布式leader节点选举

## 依赖原理

## 在ZK中添加基本节点，路径程序定义，节点类型为持久节点\(PERSISTENT\)。

## 对需要竞选leader的每个进程，在ZK中分别添加基本节点的子节点，类型为临时自编号节点\(EPHEMERAL\_SEQUENTIAL\)，并保存创建返回的实际节点路径。

## 通过delete方式删除本进程创建的子节点，可以作为退出leader状态的方式。

## 基本节点的子节点类型为临时自编号节点\(EPHEMERAL\_SEQUENTIAL\)，当进程与ZK连接中断后，ZK会自动将该节点删除，确保了断连之后其他进程对leader的选举。

## 由于ZK自编号产生的路径是递增的，因此可以通过判断基本节点的子节点中最小路径数字编号的节点是否是本进程新建的节点来判断是否获得leader地位。

## 原理图示

## 利用zk实现的分布式leader节点选举实现原理如下：

## 

## 若干进程分别尝试竞选leader，情况如下： 

## - \(1\)8个进程分别在ZK基本节点下创建临时自编号节点，获取创建成功后的实际路径 

## - \(2\)在基本节点子节点列表中，判断本进程创建节点编号是否为最小 

## - \(3\)最小编号进程获得leader地位 

## ————————————————

## 版权声明：本文为CSDN博主「johnson\_moon」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。

## 原文链接：https://blog.csdn.net/johnson\_moon/article/details/78809995



