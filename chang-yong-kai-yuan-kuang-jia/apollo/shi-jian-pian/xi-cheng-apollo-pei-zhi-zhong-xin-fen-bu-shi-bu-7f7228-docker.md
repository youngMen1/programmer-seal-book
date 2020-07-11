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

| Id | AppId | Name |
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

| NamespaceId |  | Key | Value | Comment |
| :--- | :--- | :--- | :--- | :--- |
| 1 | timeout | 100 | sample | timeout配置 |

##### 1.3.1.2.1.从别的环境导入ApolloConfigDB的项目数据

如果是全新部署的Apollo配置中心，请忽略此步。

如果不是全新部署的Apollo配置中心，比如已经使用了一段时间，这时在Apollo配置中心已经创建了不少项目以及namespace等，那么在新环境中的ApolloConfigDB中需要从其它正常运行的环境中导入必要的项目数据。

主要涉及ApolloConfigDB的下面4张表，下面同时附上需要导入的数据查询语句：

1.App

导入全部的App

如：insert into 新环境的ApolloConfigDB.App select \* from 其它环境的ApolloConfigDB.App where IsDeleted = 0;

2.AppNamespace

* 导入全部的AppNamespace
* 如：insert into 新环境的ApolloConfigDB.AppNamespace select \* from 其它环境的ApolloConfigDB.AppNamespace where IsDeleted = 0;

3.Cluster

* 导入默认的default集群
* 如：insert into 新环境的ApolloConfigDB.Cluster select \* from 其它环境的ApolloConfigDB.Cluster where Name = 'default' and IsDeleted = 0;

4.Namespace

* 导入默认的default集群中的namespace
* 如：insert into 新环境的ApolloConfigDB.Namespace select \* from 其它环境的ApolloConfigDB.Namespace where ClusterName = 'default' and IsDeleted = 0;

同时也别忘了通知用户在新的环境给自己的项目设置正确的配置信息，尤其是一些影响面比较大的公共namespace配置。

#### 1.3.1.3.调整服务端配置

Apollo自身的一些配置是放在数据库里面的，所以需要针对实际情况做一些调整。

##### 1.3.1.3.1.调整ApolloPortalDB配置

配置项统一存储在ApolloPortalDB.ServerConfig表中，也可以通过管理员工具 - 系统参数页面进行配置。

apollo.portal.envs - 可支持的环境列表

默认值是dev，如果portal需要管理多个环境的话，以逗号分隔即可（大小写不敏感），如：

`DEV,FAT,UAT,PRO`,**我这里设置为**`UAT`（**1.1.0版本增加了系统信息页面（管理员工具 -&gt;系统信息），可以通过该页面检查配置是否正确**）

##### 1.3.1.3.2.调整ApolloConfigDB配置

配置项统一存储在ApolloConfigDB.ServerConfig表中，需要注意每个环境的ApolloConfigDB.ServerConfig都需要单独配置。

1. eureka.service.url - Eureka服务Url不管是apollo-configservice还是apollo-adminservice都需要向eureka服务注册，所以需要配置eureka服务地址。 按照目前的实现，apollo-configservice本身就是一个eureka服务，所以只需要填入apollo-configservice的地址即可，如有多个，用逗号分隔（注意不要忘了/eureka/后缀）。
   **这里我填写**`http://192.168.10.138:8080/eureka`。后续文章我会单独介绍如何将Config Service和Admin Service注册到公司统一的Eureka上

### 1.3.2.获取安装包

可以通过两种方式获取安装包：

1. 直接下载安装包
   * 从GitHub Release页面下载预先打好的安装包
   * 如果对Apollo的代码没有定制需求，建议使用这种方式，可以省去本地打包的过程
2. 通过源码构建
   * 从GitHub Release页面下载Source code包或直接clone源码后在本地构建
   * 如果需要对Apollo的做定制开发，需要使用这种方式

这里我是通过源码构建的

#### 1.3.2.1.通过源码构建

到github上进行[源码下载](https://github.com/ctripcorp/apollo/releases)

##### 1.3.2.1.1.配置数据库连接信息

Apollo服务端需要知道如何连接到你前面创建的数据库，所以需要编辑scripts/build.sh，修改ApolloPortalDB和ApolloConfigDB相关的数据库连接串信息。

```
# apollo config db info
apollo_config_db_url=jdbc:mysql://192.168.10.58:3306/ApolloConfigDB?characterEncoding=utf8
apollo_config_db_username=root
apollo_config_db_password=Ibase2016

# apollo portal db info
apollo_portal_db_url=jdbc:mysql://192.168.10.58:3306/ApolloPortalDB?characterEncoding=utf8
apollo_portal_db_username=root
apollo_portal_db_password=Ibase2016
```

##### 1.3.2.1.2.配置各环境meta service地址

Apollo Portal需要在不同的环境访问不同的meta service\(apollo-configservice\)地址，所以需要在打包时提供这些信息。我这里只部署UAT环境，配置修改如下：

```
# meta server url, different environments should have different meta server addresses
#dev_meta=http://192.168.10.58:8080
#fat_meta=http://192.168.10.58:8080
uat_meta=http://192.168.10.138:8080
#pro_meta=http://192.168.10.58:8080

#META_SERVERS_OPTS="-Ddev_meta=$dev_meta -Dfat_meta=$fat_meta -Duat_meta=$uat_meta -Dpro_meta=$pro_meta"
META_SERVERS_OPTS="-Duat_meta=$uat_meta"
```

##### 1.3.2.1.3.执行编译、打包

做完上述配置后，就可以执行编译和打包了。**执行/scripts目录下build.sh脚本**，该脚本会依次打包apollo-configservice, apollo-adminservice, apollo-portal。

##### 1.3.2.1.4.获取安装包和Dockerfile文件

1.获取apollo-configservice安装包：安装包在位于apollo-configservice/target/目录下的apollo-configservice-x.x.x-github.zip，Dockerfile在apollo-configservice/src/main/docker/目录下

2.获取apollo-adminservice安装包：安装包位于apollo-adminservice/target/目录下的apollo-adminservice-x.x.x-github.zip，Dockerfile在apollo-adminservice/src/main/docker/目录下

3.获取apollo-portal安装包：安装包位于apollo-portal/target/目录下的apollo-portal-x.x.x-github.zip，Dockerfile在apollo-portal/src/main/docker/目录下

分别将上面的安装包和Dockerfile文件上传至服务器（已安装Docker）上

### 1.3.3.构建docker镜像
上传到服务器上的安胡子那个包和Dockerfile文件目录结构如下：


```

```




