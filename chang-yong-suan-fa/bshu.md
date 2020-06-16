# 1.基本介绍
B+ 树是一种树数据结构，是一个n叉树，每个节点通常有多个孩子，一颗B+树包含根节点、内部节点和叶子节点。B+ 树通常用于数据库和操作系统的文件系统中。 B+ 树的特点是能够保持数据稳定有序，其插入与修改拥有较稳定的对数时间复杂度。 B+ 树元素自底向上插入。
一个m阶的B树具有如下几个特征：

1.根结点至少有两个子女。

2.每个中间节点都包含k-1个元素和k个孩子，其中 m/2 <= k <= m

3.每一个叶子节点都包含k-1个元素，其中 m/2 <= k <= m

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
adada4999fdd48d4937f5f14c0eb7792_th.png
afffda21578b4d8a90cbdea4976fb5b6_th.png
29583d49358e41fa9c2fbc5169fb7d14_th.png
04eb120cd1e04d3a94c2482abc7deb96_th.png
3ce28ba0a2bd426ebebac9603f728603_th.png
3bd2b4220a0f4d1887e2943a729c40a1_th.png

# 2.总结

# 3.参考

B+树总结

[https://www.jianshu.com/p/71700a464e97](https://www.jianshu.com/p/71700a464e97)

B+树图文详解

[https://blog.csdn.net/qq\_26222859/article/details/80631121](https://blog.csdn.net/qq_26222859/article/details/80631121)

