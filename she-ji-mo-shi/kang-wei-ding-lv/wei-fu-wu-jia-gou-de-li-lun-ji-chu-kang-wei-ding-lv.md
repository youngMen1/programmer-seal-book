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

用通俗的说法就是：组织形式等同系统设计。

这里的系统按原作者的意思并不局限于软件系统。据说这篇文章最初投的哈佛商业评论，结果程序员屌丝的文章不入商业人士的法眼，无情被拒，康威就投到了一个编程相关的杂志，所以被误解为是针对软件开发的。最初这篇文章显然不敢自称定律（law），只是描述了作者自己的发现和总结。后来，在Brooks Law著名的人月神话中，引用这个论点，并将其“吹捧”成了现在我们熟知“康威定律”。

### 康威定律详细介绍 {#2}

Mike从他的角度归纳这篇论文中的其他一些核心观点，如下：

* 第一定律

  * Communication dictates design
  * 组织沟通方式会通过系统设计表达出来

* 第二定律

  * There is never enough time to do something right, but there is always enough time to do it over
  * 时间再多一件事情也不可能做的完美，但总有时间做完一件事情

* 第三定律

  * There is a homomorphism from the linear graph of a system to the linear graph of its design organization
  * 线型系统和线型组织架构间有潜在的异质同态特性

* 第四定律

  * The structures of large systems tend to disintegrate during development, qualitatively more so than with small systems
  * 大的系统组织总是比小系统更倾向于分解

#### 人是复杂社会动物 {#3}

* 第一定律

  * Communication dictates design
  * 组织沟通方式决定系统设计

组织的沟通和系统设计之间的紧密联系，在很多别的领域有类似的阐述。对于复杂的系统，聊设计就离不开聊人与人的沟通，解决好人与人的沟通问题，才能有一个好的系统设计。相信几乎每个程序员都读过的《人月神话》（1975年，感觉都是老古董了，经典的就是经得起时间考验）里面许多观点都和这句话有异曲同工之妙。

fb215be9fe9232ccda1fc8817b4af6e8da6abcd0.png

