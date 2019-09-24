## 微服务架构的理论基础 - 康威定律

### 概述 {#1}

关于微服务的介绍，可以参考[微服务那点事](https://yq.aliyun.com/articles/2764)。

微服务是最近非常火热的新概念，大家都在追，也都觉得很对，但是似乎没有很充足的理论基础说明这是正确的，给人的感觉是**不明觉厉**。前段时间看了Mike Amundsen[《远距离条件下的康威定律——分布式世界中实现团队构建》](https://yq.aliyun.com/go/articleRenderRedirect?url=http%3A%2F%2Fwww.infoq.com%2Fcn%2Fpresentations%2Fteam-building-implementation-in-distributed-world)（是Design RESTful API的作者）在InfoQ上的一个分享，觉得很有帮助，结合自己的一些思考，整理了该演讲的内容。

可能出乎很多人意料之外的一个事实是，微服务很多核心理念其实在半个世纪前的一篇文章中就被阐述过了，而且这篇文章中的很多论点在软件开发飞速发展的这半个世纪中竟然一再被验证，这就是[康威定律（Conway's Law）](https://yq.aliyun.com/go/articleRenderRedirect?url=http%3A%2F%2Fwww.melconway.com%2FHome%2FConways_Law.html)。

3fa834ea8c880a97d00f187f67962a8218b16f22.png

ad8ba09ed2350c2a8c2f067d80197e74aefcef8d.png

在康威的这篇文章中，最有名的一句话就是：

中文直译大概的意思就是：设计系统的组织，其产生的设计等同于组织之内、组织之间的沟通结构。看看下面的图片（来源于互联网，侵删），再想想Apple的产品、微软的产品设计，就能形象生动的理解这句话。

74ab78cb5db601e5db68adf61e6dc58f437df4e0.png

