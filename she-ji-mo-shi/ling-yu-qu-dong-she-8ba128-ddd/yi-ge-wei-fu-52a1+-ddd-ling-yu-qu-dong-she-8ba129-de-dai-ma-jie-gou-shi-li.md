# 1.一个微服务+DDD\(领域驱动设计\)的代码结构示例

## 1.1.基本介绍

领域驱动设计(DDD,Domain-Driven Design)  
DDD总体结构分为四层  :  Infrastructure(基础实施层)，Domain(领域层)，Application(应用层)，Interfaces(表示层，也叫用户界面层或是接口层)，各个层面的作用下面介绍。  
![](/static/image/994599-20180830125911190-468037055.png)  
![](/static/image/994599-20180830125945668-1072959527.png)  
对于DDD的设计而言，最重要的是如何去划分领域，划分好边界。  
在代码设计上，之前有看到过大佬用模块(Modules)来进行上下文界定和划分。如图下 :   
![](/static/image/994599-20180830131410661-290668551.png)  
而对于微服务而言，就非常适合从业务上去划分以上的各个Modules，划分好各个业务板块。

微服务 + DDD，个人觉得应该是首先是从微服务的角度\(如何划分微服务\)考虑去划分大的业务模块，每一个微服务都应该是一个可以单独部署，各司其职的模块；

而在微服务实际开发中，结合DDD的思想去划分所有属于自己的领域。

**如图示例，对于我这个Project而言，是模块已经划分好的微服务应用，代码设计上就分为  Infrastructure，Domain，Application，Interfaces : **  
![](/static/image/994599-20180830132619533-611437668.png)

### Infrastructure 层 :  基础实施层，向其他层提供通用的技术能力(比如工具类,第三方库类支持,常用基本配置,数据访问底层实现)

![](/static/image/994599-20180830134304547-660094458.png)  
![](/static/image/994599-20180830134336916-1945132941.png)

### Domain层 : 主要负责表达业务概念,业务状态信息和业务规则；是整个系统的核心层,几乎全部的业务逻辑会在该层实现。

![](/static/image/994599-20180830134410240-623245752.png)  
![](/static/image/994599-20180830134515558-56966635.png)

### Application层 :  相对于领域层,应用层是很薄的一层,应用层定义了软件要完成的任务,要尽量简单。

注 : 这里图里面所说的对内对外，对程序而言，事实上是从展现层调用应用层，应用层调用领域层，领域层或调用基础实施层。  
![](/static/image/994599-20180830134844172-1295041747.png)  
![](/static/image/994599-20180830134819652-762502148.png)

### Interfaces层 : 负责向用户显示信息和解释用户命令，请求应用层以获取用户所需要展现的数据\(比如获取首页的商品数据\)

![](/static/image/994599-20180830135806554-1845171786.png)  
![](/static/image/994599-20180830135840092-1534652017.png)  
以上，就是个人 对 微服务+DDD的代码结构示例，完整代码详见 [https://github.com/EalenXie/springcloud-microservice-ddd](https://github.com/EalenXie/springcloud-microservice-ddd)

无论我们代码结构如何规划，也并非一成不变，应该从实际出发，去思考划分结构的意义。此例子是对于微服务+DDD反应到实际开发，代码的结构设计上的一种初步的思考与探索,一个样板工程，不应该成为我们对实际DDD思考与设计的限制，本例仅供参考。

# 2.参考

可以落地的DDD到底长什么样：  
[https://www.cnblogs.com/hafiz/p/9388334.html](https://www.cnblogs.com/hafiz/p/9388334.html)  
领域驱动设计之领域模型：  
[https://www.cnblogs.com/netfocus/archive/2011/10/10/2204949.html](https://www.cnblogs.com/netfocus/archive/2011/10/10/2204949.html)  
领域驱动设计\(Domain Driven Design\)参考架构详解：  
[https://blog.csdn.net/bluishglc/article/details/6681253](https://blog.csdn.net/bluishglc/article/details/6681253)  
一个微服务+DDD\(领域驱动设计\)的代码结构示例：  
[https://www.cnblogs.com/ealenxie/p/9559781.html](https://www.cnblogs.com/ealenxie/p/9559781.html)

