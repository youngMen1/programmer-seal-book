# 1.基本介绍

B+ 树是一种树数据结构，是一个n叉树，每个节点通常有多个孩子，一颗B+树包含根节点、内部节点和叶子节点。B+ 树通常用于数据库和操作系统的文件系统中。 B+ 树的特点是能够保持数据稳定有序，其插入与修改拥有较稳定的对数时间复杂度。 B+ 树元素自底向上插入。  
一个m阶的B树具有如下几个特征：

1.根结点至少有两个子女。

2.每个中间节点都包含k-1个元素和k个孩子，其中 m/2 &lt;= k &lt;= m

3.每一个叶子节点都包含k-1个元素，其中 m/2 &lt;= k &lt;= m

4.所有的叶子结点都位于同一层。

5.每个节点中的元素从小到大排列，节点当中k-1个元素正好是k个孩子包含的元素的值域分划。  
![](/static/image/17a0c4f672b34e668a0cd2eb214c117d_th.png)  
![](/static/image/c56155c2131e45b0bf69f9ae6cba056e_th.png)  
![](/static/image/164ce3d2504c4d63945e134ca6752a2c_th.png)  
![](/static/image/891ad19fb4294e9293fdca83e8e34616_th.png)  
![](/static/image/eb790f08a02a4bcbbc7cf3f3f8a95d4d_th.png)  
![](/static/image/ff571cfd72ab4a068ce0867b0e450de8_th.png)  
![](/static/image/d4430eb5e5ef42008b1facec51636dbb_th.png)  
![](/static/image/358025867be14bb99bf8806b98e774d9_th.png)  
![](/static/image/034a86d6e1d94c798e63ab144955c0f6_th.png)  
![](/static/image/86f732dd90b74be3bf9494859fa78d66_th.png)  
![](/static/image/0611ff5a5103461e843ab627f8821419_th.png)  
![](/static/image/adada4999fdd48d4937f5f14c0eb7792_th.png)  
![](/static/image/afffda21578b4d8a90cbdea4976fb5b6_th.png)  
![](/static/image/29583d49358e41fa9c2fbc5169fb7d14_th.png)  
![](/static/image/04eb120cd1e04d3a94c2482abc7deb96_th.png)  
![](/static/image/3ce28ba0a2bd426ebebac9603f728603_th.png)  
![](/static/image/3bd2b4220a0f4d1887e2943a729c40a1_th.png)  
![](/static/image/664e36a4da0f45fcaf6e18b68d36a0b4_th.png)  
![](/static/image/514d587fa73746978200aca252837a44_th.png)  
B-树中的卫星数据（Satellite Information）：  
![](/static/image/36efa69561dc4043a17d550133e13a6c_th.png)  
![](/static/image/c3a519a9a9e8456d9be41e69709bafaf_th.png)  

B+树中的卫星数据（Satellite Information）：  
![](/static/image/d8ae1b14e9bf4b1890146eb803ee9795_th.png)  
需要补充的是，在数据库的聚集索引（Clustered Index）中，叶子节点直接包含卫星数据。在非聚集索引（NonClustered Index）中，叶子节点带有指向卫星数据的指针。  
![](/static/image/7a52624e7add4033bb49c3aa5632a681_th.png)  
![](/static/image/0ae1d08ece1e4daeac37361e86b3d6a6_th.png)  
![](/static/image/32ad0e6237624d718bb9a5346e37792e_th.png)

第一次磁盘IO：  
![](/static/image/6808907785b84be09d8c6b7c8acb5d2a_th.png)  
第二次磁盘IO：  
![](/static/image/0193eedf3a5b47129340e2b6c654ef72_th.png)  
第三次磁盘IO：  
![](/static/image/68553d369a304d798116f432247c6e3f_th.png)  
![](/static/image/3830300c15bf41f8a2c8fdf8d163fa5b_th.png)  
![](/static/image/baaed98d8fca4fb9806400651953f92d_th.png)  
![](/static/image/99d5067451ec486dbccc37611ff3747c_th.png)
![](/static/image/7522d2811b5340a7a9b222bc14ba7276_th.png)
![](/static/image/169def080e8e47a68fc4fdce3451337a_th.png)
# ..............太多了直接去B+树图文详解看

# 2.总结

## B+树的特征：

1.有k个子树的中间节点包含有k个元素（B树中是k-1个元素），每个元素不保存数据，只用来索引，所有数据都保存在叶子节点。

2.所有的叶子结点中包含了全部元素的信息，及指向含这些元素记录的指针，且叶子结点本身依关键字的大小自小而大顺序链接。

3.所有的中间节点元素都同时存在于子节点，在子节点元素中是最大（或最小）元素。

## B+树的优势：

1.单一节点存储更多的元素，使得查询的IO次数更少。

2.所有查询都要查找到叶子节点，查询性能稳定。

3.所有叶子节点形成有序链表，便于范围查询。

# 3.参考

B+树总结

[https://www.jianshu.com/p/71700a464e97](https://www.jianshu.com/p/71700a464e97)

B+树图文详解

[https://blog.csdn.net/qq\_26222859/article/details/80631121](https://blog.csdn.net/qq_26222859/article/details/80631121)

