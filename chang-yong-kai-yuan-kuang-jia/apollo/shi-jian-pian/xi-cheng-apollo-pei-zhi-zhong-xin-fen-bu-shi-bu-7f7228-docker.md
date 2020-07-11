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

* Portal部署在生产环境的机房，通过它来直接管理FAT、UAT、PRO等环境的配置

* Meta Server、Config Service和Admin Service在每个环境都单独部署，使用独立的数据库

* Meta Server、Config Service和Admin Service在生产环境部署在两个机房，实现双活

* Meta Server和Config Service部署在同一个JVM进程内，Admin Service部署在同一台服务器的另一个JVM进程内

另外可以参考下面的样例部署图：

```
为了演示方便，本文将Apollo-portal，Apollo-adminservice和Apollo-configservice部署在一台机器上
```

| 服务器 | 服务 | 端口 | 环境 |
| :--- | :--- | :--- | :--- |
| 192.168.10.138 | apollo-portal | 8070 | UAT |
| 192.168.10.138 | apollo-adminservice | 8080 | UAT |
| 192.168.10.138 | apollo-configservice | 8090 | UAT |

### 1.2.5.网络策略（来自官网）

分布式部署的时候，apollo-configservice和apollo-adminservice需要把自己的IP和端口注册到Meta Server（apollo-configservice本身）。

Apollo客户端和Portal会从Meta Server获取服务的地址（IP+端口），然后通过服务地址直接访问。

所以如果实际部署的机器有多块网卡（如docker），或者存在某些网卡的IP是Apollo客户端和Portal无法访问的（如网络安全限制），那么我们就需要在apollo-configservice和apollo-adminservice中做相关限制以避免Eureka将这些网卡的IP注册到Meta Server。

如下面这个例子就是对于apollo-configservice，把docker0和veth.\* 的网卡在注册到Eureka时忽略掉。

```
spring:
  application:
      name: apollo-configservice
  profiles:
    active: ${apollo_profile}
  cloud:
    inetutils:
      ignoredInterfaces:
        - docker0
        - veth.*
```

另外一种方式是直接指定要注册的IP，可以修改startup.sh，通过JVM System Property在运行时传入，如

`-Deureka.instance.ip-address=${指定的IP}`

，或者也可以修改apollo-adminservice或apollo-configservice 的bootstrap.yml文件，加入以下配置

```
eureka:
  instance:
    ip-address: ${指定的IP}
```

最后一种方式是直接指定要注册的IP+PORT，可以修改startup.sh，通过JVM System Property在运行时传入，如

`-Deureka.instance.homePageUrl=http://${指定的IP}:${指定的Port}`

，或者也可以修改apollo-adminservice或apollo-configservice 的bootstrap.yml文件，加入以下配置

```
eureka:
  instance:
    homePageUrl: http://${指定的IP}:${指定的Port}
    preferIpAddress: false
```

如果Apollo部署在公有云上，本地开发环境无法连接，但又需要做开发测试的话，客户端可以升级到0.11.0版本及以上，然后通过

`-Dapollo.configService=http://config-service的公网IP:端口`来跳过meta service的服务发现

## 1.3.部署步骤



