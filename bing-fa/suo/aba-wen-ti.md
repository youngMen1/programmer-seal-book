### ABA 问题

由于高并发，在CAS下，更新后可能此A非彼A。通过版本号可以解决，类似于上文Mysql 中提到的的乐观锁。

* [《Java CAS 和ABA问题》](https://www.cnblogs.com/549294286/p/3766717.html)
* [《Java 中 ABA问题及避免》](https://blog.csdn.net/li954644351/article/details/50511879)
  * AtomicStampedReference 和 AtomicStampedReference。



