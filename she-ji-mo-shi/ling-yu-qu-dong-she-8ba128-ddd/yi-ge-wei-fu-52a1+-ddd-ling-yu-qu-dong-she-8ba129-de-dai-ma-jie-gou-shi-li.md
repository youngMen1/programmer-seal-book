# 1.一个微服务+DDD(领域驱动设计)的代码结构示例

## 1.1.基本介绍
领域驱动设计(DDD,Domain-Driven Design)
DDD总体结构分为四层  :  Infrastructure(基础实施层)，Domain(领域层)，Application(应用层)，Interfaces(表示层，也叫用户界面层或是接口层)，各个层面的作用下面介绍。
![](/static/image/994599-20180830125911190-468037055.png)
![](/static/image/994599-20180830125945668-1072959527.png)
对于DDD的设计而言，最重要的是如何去划分领域，划分好边界。
在代码设计上，之前有看到过大佬用模块(Modules)来进行上下文界定和划分。如图下 : 
![](/static/image/994599-20180830131410661-290668551.png)
而对于微服务而言，就非常适合从业务上去划分以上的各个Modules，划分好各个业务板块。

微服务 + DDD，个人觉得应该是首先是从微服务的角度(如何划分微服务)考虑去划分大的业务模块，每一个微服务都应该是一个可以单独部署，各司其职的模块；

而在微服务实际开发中，结合DDD的思想去划分所有属于自己的领域。

如图示例，对于我这个Project而言，是模块已经划分好的微服务应用，代码设计上就分为  Infrastructure，Domain，Application，Interfaces : 
![](/static/image/994599-20180830132619533-611437668.png)
Infrastructure 层 :  基础实施层，向其他层提供通用的技术能力(比如工具类,第三方库类支持,常用基本配置,数据访问底层实现)
# 2.参考
可以落地的DDD到底长什么样：
https://www.cnblogs.com/hafiz/p/9388334.html
领域驱动设计之领域模型：
https://www.cnblogs.com/netfocus/archive/2011/10/10/2204949.html
领域驱动设计(Domain Driven Design)参考架构详解：
https://blog.csdn.net/bluishglc/article/details/6681253
一个微服务+DDD(领域驱动设计)的代码结构示例：
https://www.cnblogs.com/ealenxie/p/9559781.html

