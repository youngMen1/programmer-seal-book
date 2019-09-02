# Summary

* [Introduction](README.md)
* [数据结构](shu-ju-jie-gou.md)
  * [队列](shu-ju-jie-gou/dui-lie.md)
* [常用算法](chang-yong-suan-fa.md)
  * [排序、查找算法](chang-yong-suan-fa/pai-xu-3001-cha-zhao-suan-fa.md)
    * [选择排序](chang-yong-suan-fa/pai-xu-3001-cha-zhao-suan-fa/xuan-ze-pai-xu.md)
    * 插入排序
    * 快速排序
    * 归并排序
    * 希尔排序
    * 堆排序
    * 计数排序
    * 桶排序
    * 基数排序
    * 二分查找
    * java中的排序工具
  * 布隆过滤器
* [并发](bing-fa.md)
  * [Java 并发](bing-fa/java-bing-fa.md)
    * [并发的优缺点](bing-fa/java-bing-fa/bing-fa-de-you-que-dian.md)
    * [线程状态转换以及基本操作](bing-fa/java-bing-fa/xian-cheng-zhuang-tai-zhuan-huan-yi-ji-ji-ben-cao-zuo.md)
    * [Java内存模型以及happens-before](bing-fa/java-bing-fa/javanei-cun-mo-xing-yi-ji-happens-before.md)
    * [java关键字---synchronized](bing-fa/java-bing-fa/javaguan-jian5b57-synchronized.md)
    * [java关键字---volatile](bing-fa/java-bing-fa/javaguan-jian-5b57-volatile.md)
    * [java关键字--final](bing-fa/java-bing-fa/javaguan-jian-5b57-final.md)
    * [三大性质总结：原子性、可见性以及有序性](bing-fa/java-bing-fa/san-da-xing-zhi-zong-jie-ff1a-yuan-zi-xing-3001-ke-jian-xing-yi-ji-you-xu-xing.md)
    * [初识Lock与AbstractQueuedSynchronizer\(AQS\)](bing-fa/java-bing-fa/chu-shi-lock-yu-abstractqueuedsynchronizer-aqs.md)
    * [深入理解AbstractQueuedSynchronizer\(AQS\)](bing-fa/java-bing-fa/shen-ru-li-jie-abstractqueuedsynchronizer-aqs.md)
    * [彻底理解ReentrantLock](bing-fa/java-bing-fa/che-di-li-jie-reentrantlock.md)
    * [深入理解读写锁ReentrantReadWriteLock](bing-fa/java-bing-fa/shen-ru-li-jie-du-xie-suo-reentrantreadwritelock.md)
    * [详解Condition的await和signal等待通知机制](bing-fa/java-bing-fa/xiang-jie-condition-de-await-he-signal-deng-dai-tong-zhi-ji-zhi.md)
    * [LockSupport的工具](bing-fa/java-bing-fa/locksupportde-gong-ju.md)
    * [并发容器之ConcurrentHashMap\(JDK 1.8版本\)](bing-fa/java-bing-fa/bing-fa-rong-qi-zhi-concurrenthashmap-jdk-1-8-ban-672c29.md)
    * [并发容器之ConcurrentLinkedQueue](bing-fa/java-bing-fa/bing-fa-rong-qi-zhi-concurrentlinkedqueue.md)
    * [并发容器之CopyOnWriteArrayList](bing-fa/java-bing-fa/bing-fa-rong-qi-zhi-copyonwritearraylist.md)
    * [并发容器之ThreadLocal](bing-fa/java-bing-fa/bing-fa-rong-qi-zhi-threadlocal.md)
    * [一篇文章，从源码深入详解ThreadLocal内存泄漏问题](bing-fa/java-bing-fa/yi-pian-wen-zhang-ff0c-cong-yuan-ma-shen-ru-xiang-jie-threadlocal-nei-cun-xie-lou-wen-ti.md)
    * [并发容器之BlockingQueue](bing-fa/java-bing-fa/bing-fa-rong-qi-zhi-blockingqueue.md)
    * [并发容器之ArrayBlockingQueue和LinkedBlockingQueue实现原理详解](bing-fa/java-bing-fa/bing-fa-rong-qi-zhi-arrayblockingqueue-helinkedblockingqueue-shi-xian-yuan-li-xiang-jie.md)
    * [线程池ThreadPoolExecutor实现原理](bing-fa/java-bing-fa/xian-cheng-chi-threadpoolexecutor-shi-xian-yuan-li.md)
    * [线程池之ScheduledThreadPoolExecutor](bing-fa/java-bing-fa/xian-cheng-chi-zhi-scheduledthreadpoolexecutor.md)
    * [FutureTask基本操作总结](bing-fa/java-bing-fa/futuretaskji-ben-cao-zuo-zong-jie.md)
    * Java中atomic包中的原子操作类总结
  * [多线程](bing-fa/duo-xian-cheng.md)
* [操作系统](cao-zuo-xi-tong.md)
  * 计算机原理
* [设计模式](she-ji-mo-shi.md)
  * [设计模式的六大原则](she-ji-mo-shi/she-ji-mo-shi-de-liu-da-yuan-ze.md)
* [运维 & 统计 & 技术支持](yun-wei-and-tong-ji-and-ji-zhu-zhi-chi.md)
  * [常规监控](yun-wei-and-tong-ji-and-ji-zhu-zhi-chi/chang-gui-jian-kong.md)
* [中间件](zhong-jian-jian.md)
  * [Web Server](zhong-jian-jian/web-server.md)
    * [Nginx](zhong-jian-jian/web-server/nginx.md)
      * [Ngnix的基本学习-多进程和Apache的比较](zhong-jian-jian/web-server/nginx/ngnixde-ji-ben-xue-4e60-duo-jin-cheng-he-apache-de-bi-jiao.md)
      * [Nginx与Apache的对比以及优缺点](zhong-jian-jian/web-server/nginx/nginxyu-apache-de-dui-bi-yi-ji-you-que-dian.md)
    * [OpenResty](zhong-jian-jian/web-server/openresty.md)
    * Tengine
    * Apache Httpd
    * Tomcat
    * [Jetty](zhong-jian-jian/web-server/jetty.md)
      * [Jetty 的工作原理以及与 Tomcat 的比较](zhong-jian-jian/web-server/jetty/jetty-de-gong-zuoyuan-li-yi-ji-yu-tomcat-de-bi-jiao.md)
      * [jetty和tomcat优势比较](zhong-jian-jian/web-server/jetty/jettyhe-tomcat-you-shi-bi-jiao.md)
    * [本地缓存](zhong-jian-jian/huan-cun/ben-di-huan-cun.md)
      * HashMap本地缓存
  * [缓存](zhong-jian-jian/huan-cun.md)
    * [缓存失效策略（FIFO 、LRU、LFU三种算法的区别）](zhong-jian-jian/huan-cun/huan-cun-shi-xiao-ce-lve-ff08-fifo-lru-lfu-san-zhong-suan-fa-de-qu-bie-ff09.md)
  * [客户端缓存](zhong-jian-jian/ke-hu-duan-huan-cun.md)
    * 浏览器端缓存
    * H5 和移动端 WebView 缓存机制解析与实战
  * [服务端缓存](zhong-jian-jian/fu-wu-duan-huan-cun.md)
    * [Memcached](zhong-jian-jian/fu-wu-duan-huan-cun/memcached.md)
    * [Redis](zhong-jian-jian/fu-wu-duan-huan-cun/redis.md)
    * [Web缓存](zhong-jian-jian/webhuan-cun.md)
    * [Tair](zhong-jian-jian/fu-wu-duan-huan-cun/tair.md)
  * [定时调度](zhong-jian-jian/ding-shi-diao-du.md)
    * [单机定时调度](zhong-jian-jian/ding-shi-diao-du/dan-ji-ding-shi-diao-du.md)
    * [分布式定时调度](zhong-jian-jian/ding-shi-diao-du/fen-bu-shi-ding-shi-diao-du.md)
  * [RPC](zhong-jian-jian/rpc.md)
  * 数据库中间件
  * 日志系统
  * [配置中心](zhong-jian-jian/pei-zhi-zhong-xin.md)
    * Apollo - 携程开源的配置中心应用
    * 基于zookeeper实现统一配置管理
    * Spring Cloud Config 分布式配置中心使用教程
    * servlet3.0 新特性——异步处理
* [网络](wang-luo.md)
  * [协议](wang-luo/xie-yi.md)
    * [OSI 七层协议](wang-luo/xie-yi/osi-qi-ceng-xie-yi.md)
    * [TCP/IP](wang-luo/xie-yi/tcpip.md)
      * [TCP协议中的三次握手和四次挥手](wang-luo/xie-yi/tcpip/tcpxie-yi-zhong-de-san-ci-wo-shou-he-si-ci-hui-shou.md)
    * [HTTP](wang-luo/xie-yi/http.md)
    * [HTTP2.0](wang-luo/xie-yi/http20.md)
      * [HTTP2.0的基本单位为二进制帧](wang-luo/xie-yi/http20/http20de-ji-ben-dan-wei-wei-er-jin-zhi-zheng.md)
    * [HTTPS](wang-luo/xie-yi/https.md)
      * [https原理通俗了解](wang-luo/xie-yi/https/httpsyuan-li-tong-su-le-jie.md)
      * [八大免费SSL证书-给你的网站免费添加Https安全加密](wang-luo/xie-yi/https/ba-da-mian-fei-ssl-zheng-4e66-gei-ni-de-wang-zhan-mian-fei-tian-jia-https-an-quan-jia-mi.md)
  * [网络模型](wang-luo/wang-luo-mo-xing.md)
    * [web优化必须了解的原理之I/o的五种模型和web的三种工作模式](wang-luo/wang-luo-mo-xing/webyou-hua-bi-xu-le-jie-de-yuan-li-zhi-i-o-de-wu-zhong-mo-xing-he-web-de-san-zhong-gong-zuo-mo-shi.md)
    * [select、poll、epoll之间的区别总结\[整理\]](wang-luo/wang-luo-mo-xing/selectpollepollzhi-jian-de-qu-bie-zong-7ed35b-zheng-74065d.md)
    * [select，poll，epoll优缺点及比较](wang-luo/wang-luo-mo-xing/selectpollepollyou-que-dian-ji-bi-jiao.md)
    * [深入理解Java NIO](wang-luo/wang-luo-mo-xing/shen-ru-li-jie-java-nio.md)
    * [BIO与NIO、AIO的区别\(这个容易理解\)](wang-luo/wang-luo-mo-xing/bioyu-nio-aio-de-qu-522b28-zhe-ge-rong-yi-li-89e329.md)
    * [两种高效的服务器设计模型：Reactor和Proactor模型](wang-luo/wang-luo-mo-xing/liang-zhong-gao-xiao-de-fu-wu-qi-she-ji-mo-xing-ff1a-reactor-he-proactor-mo-xing.md)
    * [Epoll](wang-luo/wang-luo-mo-xing/epoll.md)
    * [kqueue](wang-luo/kqueue.md)
      * [kqueue用法简介](wang-luo/kqueue/kqueueyong-fa-jian-jie.md)
  * [连接和短连接](wang-luo/lian-jie-he-duan-lian-jie.md)
    * [TCP/IP系列——长连接与短连接的区别](wang-luo/lian-jie-he-duan-lian-jie/tcpipxi-lie-2014-2014-chang-lian-jie-yu-duan-lian-jie-de-qu-bie.md)
  * [框架](wang-luo/kuang-jia.md)
    * [Netty原理剖析](wang-luo/kuang-jia/nettyyuan-li-pou-xi.md)
  * [零拷贝（Zero-copy）](wang-luo/ling-kaobei-ff08-zero-copy.md)
    * [对于 Netty ByteBuf 的零拷贝\(Zero Copy\) 的理解](wang-luo/ling-kaobei-ff08-zero-copy/dui-yu-netty-bytebuf-de-ling-kao-8d1d28-zero-copy-de-li-jie.md)
  * [序列化\(二进制协议\)](wang-luo/xu-lie-531628-er-jin-zhi-xie-8bae29.md)
* [数据库](shu-ju-ku.md)
  * [基础理论](shu-ju-ku/ji-chu-li-lun.md)
    * [关系数据库设计的三大范式](shu-ju-ku/ji-chu-li-lun/guan-xi-shu-ju-ku-she-ji-de-san-da-fan-shi.md)
      * [据库的三大范式以及五大约束](shu-ju-ku/ji-chu-li-lun/guan-xi-shu-ju-ku-she-ji-de-san-da-fan-shi/ju-ku-de-san-da-fan-shi-yi-ji-wu-da-yue-shu.md)
  * [MySQL](shu-ju-ku/mysql.md)
    * [原理](shu-ju-ku/mysql/yuan-li.md)
      * [MySQL的InnoDB索引原理详解](shu-ju-ku/mysql/yuan-li/mysqlde-innodb-suo-yin-yuan-li-xiang-jie.md)
      * [MySQL存储引擎－－MyISAM与InnoDB区别](shu-ju-ku/mysql/yuan-li/mysqlcun-chu-yin-qing-ff0d-ff0d-myisam-yu-innodb-qu-bie.md)
      * [myisam和innodb索引实现的不同](shu-ju-ku/mysql/yuan-li/myisamhe-innodb-suo-yin-shi-xian-de-bu-tong.md)
    * [InnoDB](shu-ju-ku/mysql/innodb.md)
      * [『浅入浅出』MySQL 和 InnoDB](shu-ju-ku/mysql/innodb/300e-qian-ru-qianchu-300f-mysql-he-innodb.md)
  * [NoSQL](shu-ju-ku/nosql.md)
* [搜索引擎](sou-suo-yin-qing.md)
  * [搜索引擎原理](sou-suo-yin-qing/sou-suo-yin-qing-yuan-li.md)
* [性能](xing-neng.md)
  * [性能优化方法论](xing-neng/xing-neng-you-hua-fang-fa-lun.md)
* [大数据](da-shu-ju.md)
  * 流式计算
* [安全](an-quan.md)
  * [web 安全](an-quan/web-an-quan.md)
    * [XSS](an-quan/web-an-quan/xss.md)
      * [xss攻击原理与解决方法](an-quan/web-an-quan/xss/xssgong-ji-yuan-li-yu-jie-jue-fang-fa.md)
    * [CSRF](an-quan/web-an-quan/csrf.md)
      * [CSRF原理及防范](an-quan/web-an-quan/csrf/csrfyuan-li-ji-fang-fan.md)
    * [SQL 注入](an-quan/web-an-quan/sql-zhu-ru.md)
      * [SQL 注入](an-quan/web-an-quan/sql-zhu-ru/sql-zhu-ru.md)
    * [Hash Dos](an-quan/web-an-quan/hash-dos.md)
    * [脚本注入](an-quan/web-an-quan/jiao-ben-zhu-ru.md)
      * [上传文件漏洞原理及防范](an-quan/web-an-quan/jiao-ben-zhu-ru/shang-chuan-wen-jian-lou-dong-yuan-li-ji-fang-fan.md)
    * 漏洞扫描工具
    * 验证码
    * DDoS 防范
  * 用户隐私信息保护
  * 序列化漏洞
* [常用开源框架](chang-yong-kai-yuan-kuang-jia.md)
  * 开源协议
* [分布式设计](fen-bu-shi-she-ji.md)
  * 扩展性设计
* [设计思想 & 开发模式](she-ji-si-xiang-and-kai-fa-mo-shi.md)
  * DDD\(Domain-driven Design - 领域驱动设计\)
* [项目管理](xiang-mu-guan-li.md)
  * 架构评审

