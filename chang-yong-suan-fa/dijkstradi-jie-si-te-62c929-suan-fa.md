# 1.Dijkstra算法

Dijkstra算法算是贪心思想实现的，首先把起点到所有点的距离存下来找个最短的，然后松弛一次再找出最短的，所谓的松弛操作就是，遍历一遍看通过刚刚找到的距离最短的点作为中转站会不会更近，如果更近了就更新距离，这样把所有的点找遍之后就存下了起点到其他所有点的最短距离。

## 1.1.引入问题

指定一个点（源点）到其余各个顶点的最短路径，也叫做“单源最短路径”。例如求下图中的1号顶点到2、3、4、5、6号顶点的最短路径。

20181120091021734.png









# 2.参考

Dijkstra算法图文详解：[https://blog.csdn.net/lbperfect123/article/details/84281300](https://blog.csdn.net/lbperfect123/article/details/84281300)
