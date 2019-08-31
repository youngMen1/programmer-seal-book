### Nginx

* [《Ngnix的基本学习-多进程和Apache的比较》](https://blog.csdn.net/qq_25797077/article/details/52200722)

  * Nginx 通过异步非阻塞的事件处理机制实现高并发。Apache 每个请求独占一个线程，非常消耗系统资源。
  * 事件驱动适合于IO密集型服务\(Nginx\)，多进程或线程适合于CPU密集型服务\(Apache\)，所以Nginx适合做反向代理，而非web服务器使用。

* [《nginx与Apache的对比以及优缺点》](https://www.cnblogs.com/cunkouzh/p/5410154.html)

  * nginx只适合静态和反向代理，不适合处理动态请求。



