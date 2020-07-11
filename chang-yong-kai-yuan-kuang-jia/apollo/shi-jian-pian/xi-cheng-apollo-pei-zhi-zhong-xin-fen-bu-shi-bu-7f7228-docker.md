# 1.携程 Apollo 配置中心分布式部署\(Docker\)

## 1.1.基本介绍

在[Spring Boot 2.0 整合携程Apollo配置中心](https://blog.csdn.net/AaronSimon/article/details/83657612)一文中，我们在本地快速部署试用了Apollo。本文将介绍如何按照分布式部署（采用Docker部署）的方式编译、打包、部署Apollo配置中心，从而可以在开发、测试、生产等环境分别部署运行。

## 1.2.准备工作

本文将在CentOS 7.x上部署Apollo配置中心服务端。

### 1.1 Java和MySQ

对于Java和MySQL的要求可以参考[Spring Boot 2.0 整合携程Apollo配置中心](https://blog.csdn.net/AaronSimon/article/details/83657612)准备工作的部分。

### 1.2 Docker环境安装

对于Docker环境的安装可以参考[CentOS 7.x 安装Docker](https://blog.csdn.net/AaronSimon/article/details/82711512)

### 1.3 Docker Compose安装

对于Docker Compose的安装可以参考[Docker Compose编排微服务](https://blog.csdn.net/AaronSimon/article/details/82711595)

### 1.4 部署策略\(来自官网\)

分布式部署需要事先确定部署的环境以及部署方式。Apollo目前支持以下环境：

* DEV 开发环境
* FAT 测试环境，相当于alpha环境\(功能测试\)
* UAT 集成环境，相当于beta环境（回归测试）
* PRO 生产环境
 
   如果希望添加自定义的环境名称，具体步骤可以参考
  [部署&开发遇到的常见问题\#42-添加自定义的环境](https://github.com/ctripcorp/apollo/wiki/%E9%83%A8%E7%BD%B2&%E5%BC%80%E5%8F%91%E9%81%87%E5%88%B0%E7%9A%84%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98#42-%E6%B7%BB%E5%8A%A0%E8%87%AA%E5%AE%9A%E4%B9%89%E7%9A%84%E7%8E%AF%E5%A2%83)

以ctrip为例，其部署策略如下：

  


  


作者：AaronSimon

  


链接：https://www.jianshu.com/p/039e7ca8ad0a

  


来源：简书

  


著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

