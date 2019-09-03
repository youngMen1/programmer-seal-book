### 平滑启动

* 平滑重启应用思路 1.端流量（如vip层）、2. flush 数据\(如果有\)、3, 重启应用

* [《JVM安全退出（如何优雅的关闭java服务）》](https://blog.csdn.net/u011001084/article/details/73480432)推荐推出方式：System.exit，Kill SIGTERM；不推荐 kill-9；用 Runtime.addShutdownHook 注册钩子。

* [《常见Java应用如何优雅关闭》](http://ju.outofmemory.cn/entry/337235)Java、Spring、Dubbo 优雅关闭方式。



