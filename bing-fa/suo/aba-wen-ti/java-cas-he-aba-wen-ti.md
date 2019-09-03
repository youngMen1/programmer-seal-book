## [Java CAS 和ABA问题](https://www.cnblogs.com/549294286/p/3766717.html)

2014-06-03 23:42

[Loull](https://www.cnblogs.com/549294286/)

阅读\(

29341

\) 评论\(

4

\)

[编辑](https://i.cnblogs.com/EditPosts.aspx?postid=3766717)

[收藏](javascript:void%280%29)

独占锁：是一种悲观锁，synchronized就是一种独占锁，会导致其它所有需要锁的线程挂起，等待持有锁的线程释放锁。

乐观锁：每次不加锁，假设没有冲突去完成某项操作，如果因为冲突失败就重试，直到成功为止。

**一、CAS 操作**

乐观锁用到的机制就是CAS，Compare and Swap。

CAS有3个操作数，内存值V，旧的预期值A，要修改的新值B。当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做。

**1、非阻塞算法 （nonblocking algorithms）**

