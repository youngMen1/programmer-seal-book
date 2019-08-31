## 定时调度

### 单机定时调度

* [《linux定时任务cron配置》](https://www.cnblogs.com/shuaiqing/p/7742382.html)

* [《Linux cron运行原理》](https://my.oschina.net/daquan/blog/483305)

  * fork 进程 + sleep 轮询

* [《Quartz使用总结》](https://www.cnblogs.com/drift-ice/p/3817269.html)

* [《Quartz源码解析 ---- 触发器按时启动原理》](https://blog.csdn.net/wenniuwuren/article/details/42082981/)

* [《quartz原理揭秘和源码解读》](https://www.jianshu.com/p/bab8e4e32952)

  * 定时调度在 QuartzSchedulerThread 代码中，while\(\)无限循环，每次循环取出时间将到的trigger，触发对应的job，直到调度器线程被关闭。

### 分布式定时调度

* [《这些优秀的国产分布式任务调度系统，你用过几个？》](https://blog.csdn.net/qq_16216221/article/details/70314337)

  * opencron、LTS、XXL-JOB、Elastic-Job、Uncode-Schedule、Antares

* [《Quartz任务调度的基本实现原理》](https://www.cnblogs.com/zhenyuyaodidiao/p/4755649.html)

  * Quartz集群中，独立的Quartz节点并不与另一其的节点或是管理节点通信，而是通过相同的数据库表来感知到另一Quartz应用的

* [《Elastic-Job-Lite 源码解析》](http://www.iocoder.cn/categories/Elastic-Job-Lite/?vip&architect-awesome)

* [《Elastic-Job-Cloud 源码解析》](http://www.iocoder.cn/categories/Elastic-Job-Cloud/?vip&architect-awesome)



