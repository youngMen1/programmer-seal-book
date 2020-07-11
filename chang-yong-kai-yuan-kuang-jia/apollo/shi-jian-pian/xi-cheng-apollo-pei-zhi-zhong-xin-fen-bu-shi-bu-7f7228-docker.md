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

部署步骤共三步：

1. 创建数据库：Apollo服务端依赖于MySQL数据库，所以需要事先创建并完成初始化
2. 获取安装包：通过源码构建
3. 构建docker镜像：为apollo-configservice, apollo-adminservice, apollo-portal构建Docker镜像
4. 部署Apollo服务端：构建镜像后通过docker compose就可以部署到公司的测试和生产环境了

### 1.3.1.创建数据库

Apollo服务端共需要两个数据库：ApolloPortalDB和ApolloConfigDB，官网把数据库、表的创建和样例数据都分别准备了sql文件（在下载的源码`/scripts/sql`目录下），只需要导入数据库即可。

由于我只在一台服务器上做演示，所以`ApolloPortalDB`和`ApolloConfigDB`在UAT环境部署一套就可以了

| 服务器 | 数据库 | 端口 | 环境 |
| :--- | :--- | :--- | :--- |
| 192.168.10.58 | ApolloPortalDB | 3306 | UAT |
| 192.168.10.58 | ApolloConfigDB | 3306 | UAT |

#### 1.3.1.1.创建ApolloPortalDB

通过各种Mysql客户端（Navicat,DataGrip等）导入sql/apolloportaldb.sql即可  
下面以MySQL原生客户端为例：

```
source /your_local_path/sql/apolloportaldb.sql
```

导入成功后，可以通过执行以下sql语句来验证：

    select `Id`, `AppId`, `Name` from ApolloPortalDB.App;

| Id |
| :--- |


|  | AppId | Name |
| :--- | :--- | :--- |
| 1 | SampleApp | Sample App |

#### 1.3.1.2.创建ApolloConfigDB

通过各种Mysql客户端（Navicat,DataGrip等）导入

`sql/apolloconfigdb.sql`

即可

下面以MySQL原生客户端为例：

```
source /your_local_path/sql/apolloconfigdb.sql
```

导入成功后，可以通过执行以下sql语句来验证：

    select `NamespaceId`, `Key`, `Value`, `Comment` from ApolloConfigDB.Item;

| NamespaceId |
| :--- |


|  |  | Key | Value | Comment |
| :--- | :--- | :--- | :--- | :--- |
| 1 | timeout | 100 | sample | timeout配置 |

##### 1.3.1.2.1.从别的环境导入ApolloConfigDB的项目数据

如果是全新部署的Apollo配置中心，请忽略此步。

如果不是全新部署的Apollo配置中心，比如已经使用了一段时间，这时在Apollo配置中心已经创建了不少项目以及namespace等，那么在新环境中的ApolloConfigDB中需要从其它正常运行的环境中导入必要的项目数据。

主要涉及ApolloConfigDB的下面4张表，下面同时附上需要导入的数据查询语句：

  


  






