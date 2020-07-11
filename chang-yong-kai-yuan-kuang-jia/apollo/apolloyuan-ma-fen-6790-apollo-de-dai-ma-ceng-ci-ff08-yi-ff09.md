# 1.Apollo源码分析-Apollo的代码层次(三)

## 1.1.基本介绍

不同与其它中间件框架，Apollo中有大量的业务代码，它向我们展示了大神是如何写业务代码的：maven依赖的层次结构，如何进行基础包配置，以及工具类编写，可以称之为springboot之最佳实践。

## 1.2.apollo项目依赖

apollo中有7个子项目
最重要的有四个
apollo-portal:后台管理服务
apollo-admin:后台配置管理服务，用户发布了的配置项会经过portal->admin更新到数据库
apollo-configservice: 配置管理服务，客户端通过该服务拉取配置项
apollo-client:客户端，集成该客户端拉取配置项
此外还有apollo-biz，apollo-common,apollo-core提供基础服务

其依赖关系如下:
![](/static/image/2051242107-5cf737fd43e95_articlex.jpg)


### apollo-common分析

![](/static/image/304542458-5cf739653eaa7_articlex.jpg)














