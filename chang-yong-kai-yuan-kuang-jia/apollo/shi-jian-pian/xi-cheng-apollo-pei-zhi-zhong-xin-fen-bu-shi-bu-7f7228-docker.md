# 1.携程 Apollo 配置中心分布式部署\(Docker\)

## 1.1.基本介绍

在[Spring Boot 2.0 整合携程Apollo配置中心](https://blog.csdn.net/AaronSimon/article/details/83657612)一文中，我们在本地快速部署试用了Apollo。本文将介绍如何按照分布式部署（采用Docker部署）的方式编译、打包、部署Apollo配置中心，从而可以在开发、测试、生产等环境分别部署运行。

## 1.2.准备工作

本文将在CentOS 7.x上部署Apollo配置中心服务端。

### 1.2.1.Java和MySQ

对于Java和MySQL的要求可以参考[Spring Boot 2.0 整合携程Apollo配置中心](https://blog.csdn.net/AaronSimon/article/details/83657612)准备工作的部分。

### 1.2.2.Docker环境安装

对于Docker环境的安装可以参考[CentOS 7.x 安装Docker](https://blog.csdn.net/AaronSimon/article/details/82711512)

### 1.2.3.Docker Compose安装

对于Docker Compose的安装可以参考[Docker Compose编排微服务](https://blog.csdn.net/AaronSimon/article/details/82711595)

### 1.2.4.部署策略\(来自官网\)

分布式部署需要事先确定部署的环境以及部署方式。Apollo目前支持以下环境：

* DEV 开发环境
* FAT 测试环境，相当于alpha环境\(功能测试\)
* UAT 集成环境，相当于beta环境（回归测试）
* PRO 生产环境如果希望添加自定义的环境名称，具体步骤可以参考  
  [部署&开发遇到的常见问题\#42-添加自定义的环境](https://github.com/ctripcorp/apollo/wiki/部署&开发遇到的常见问题#42-添加自定义的环境)

以ctrip为例，其部署策略如下：
![](/static/image/5389623-7342136aa7097287.webp)


