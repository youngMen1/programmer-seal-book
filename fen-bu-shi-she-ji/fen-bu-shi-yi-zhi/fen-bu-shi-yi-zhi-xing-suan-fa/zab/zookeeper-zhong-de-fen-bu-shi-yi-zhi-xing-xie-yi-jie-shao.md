# Zookeeper 中的分布式一致性协议介绍

# 背景

在分布式系统中实现一致性是件有挑战的事。经典的二阶段提交、三阶段提交都不能完美的解决这一问题，有关传统的的分布式系统一致性问题可以看[这里](https://link.jianshu.com?t=http://coolshell.cn/articles/10910.html)。[Paxos](https://link.jianshu.com?t=https://en.wikipedia.org/wiki/Paxos_%28computer_science%29) 算法能完美地达到分布式系统的一致性，但由于较为复杂，在实际工程上不是很合适，Zab 协议借鉴了 Paxos 的思想，并进行了改进，以满足工程上的实际需求。

作者：两棵橘树

链接：[https://www.jianshu.com/p/fb527a64deee](https://www.jianshu.com/p/fb527a64deee)

来源：简书

简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。

