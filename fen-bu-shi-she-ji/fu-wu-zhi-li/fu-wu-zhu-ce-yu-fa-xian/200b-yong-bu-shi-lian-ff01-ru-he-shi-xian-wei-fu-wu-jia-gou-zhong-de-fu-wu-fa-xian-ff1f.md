![](http://mmbiz.qpic.cn/mmbiz/CiaJxoTn5FwrPtaEibEr6eHvJVuVhviaTic7xdW7Dzu3qaWKxVyLZnicBr2yLeoicuxiapt8TtQ8oKQdbn0iaFNyaygjmg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**为什么使用服务发现？**

![](http://mmbiz.qpic.cn/mmbiz/CiaJxoTn5FwrPtaEibEr6eHvJVuVhviaTic7AYMJNqNJSt27sMmIUib7KUCOib0ltrwVOoeJFDqybygPz08EbU1dwZpw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

想象一下，如果你在写代码调用一个有REST API或Thrift API的服务，你的代码需要知道一个服务实例的网络地址（IP地址和端口）。运行在物理硬件上的传统应用中，服务实例的网络地址是相对静态的，你的代码可以从一个很少更新的配置文件中读取网络地址。

  


在一个现代的，基于云的微服务应用中，这个问题就变得复杂多了，如下图所示：

