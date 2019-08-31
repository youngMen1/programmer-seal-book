### Memcached

* [《Memcached 教程》](http://www.runoob.com/Memcached/Memcached-tutorial.html)

* [《深入理解Memcached原理》](https://blog.csdn.net/chenleixing/article/details/47035453)

  * 采用多路复用技术提高并发性。
  * slab分配算法： memcached给Slab分配内存空间，默认是1MB。分配给Slab之后 把slab的切分成大小相同的chunk，Chunk是用于缓存记录的内存空间，Chunk 的大小默认按照1.25倍的速度递增。好处是不会频繁申请内存，提高IO效率，坏处是会有一定的内存浪费。

* [《Memcached软件工作原理》](https://www.jianshu.com/p/36e5cd400580)

* [《Memcache技术分享：介绍、使用、存储、算法、优化、命中率》](http://zhihuzeye.com/archives/2361)

* [《memcache 中 add 、 set 、replace 的区别》](https://blog.csdn.net/liu251890347/article/details/37690045)

  * 区别在于当key存在还是不存在时，返回值是true和false的。

* [**《memcached全面剖析》**](https://pan.baidu.com/s/1qX00Lti?errno=0&errmsg=Auth%20Login%20Sucess&&bduss=&ssnerror=0&traceid=)



