# 1.Spring Boot 2.0 整合携程Apollo配置中心

## 1.1.基本介绍

Apollo（阿波罗）是携程框架部门研发的分布式配置中心，能够集中化管理应用不同环境、不同集群的配置，配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性，适用于微服务配置管理场景。

服务端基于Spring Boot和Spring Cloud开发，打包后可以直接运行，不需要额外安装Tomcat等应用容器。

Java客户端不依赖任何框架，能够运行于所有Java运行时环境，同时对Spring/Spring Boot环境也有较好的支持。

.Net客户端不依赖任何框架，能够运行于所有.Net运行时环境。

如果想要深入了解，可以到github上参见[Apollo配置中心](https://github.com/ctripcorp/apollo#screenshots)，官网的介绍很详细。本章主要讲述Spring Boot 2.0 整合Apollo配置中心。

## 1.2.Apollo配置中心服务端\(来源于官网\)

本文的重点在于Apollo在客户端的使用，所以Apollo服务端使用的是官网提供的 Quick Start（针对本地测试使用），后续文章会专门讲述Apollo服务端在分布式环境下的部署。

### 1.2.1.准备工作

1.Java  
Apollo服务端要求Java 1.8+，客户端要求Java 1.7+，笔者本地使用的是Java 1.8。  
2.MySQL  
Apollo的表结构对timestamp使用了多个default声明，所以需要5.6.5以上版本。笔者本地使用的是8.0.13版本  
3.下载 Quick Start

官网为我们准备了 Quick Start 安装包。大家只需要下载到本地，就可以直接使用，免去了编译、打包过程。大家可以到[github下载](https://github.com/nobodyiam/apollo-build-scripts)

，也可以通过[百度云盘下载](https://pan.baidu.com/s/187W86LoeVuv3DMrOJhcg1A)

### 1.2.2.安装步骤

#### 1.2.2.1 创建数据库

Apollo服务端共需要两个数据库：ApolloPortalDB和ApolloConfigDB，官网把数据库、表的创建和样例数据都分别准备了sql文件（在下载的 Quick Start 安装包的sql目录下），只需要导入数据库即可。

##### 1.2.2.1.1 创建ApolloPortalDB

通过各种Mysql客户端（Navicat,DataGrip等）导入sql/apolloportaldb.sql即可  
下面以MySQL原生客户端为例：

`source /your_local_path/sql/apolloportaldb.sql`  
导入成功后，可以通过执行以下sql语句来验证：

    select `Id`, `AppId`, `Name` from ApolloPortalDB.App;

| Id | AppId | Name |
| :--- | :--- | :--- |
| 1 | SampleApp | Sample App |

#### 1.2.2.1.2.创建ApolloConfigDB

通过各种Mysql客户端（Navicat,DataGrip等）导入`sql/apolloconfigdb.sql`即可  
下面以MySQL原生客户端为例：

```
source /your_local_path/sql/apolloconfigdb.sql
```

导入成功后，可以通过执行以下sql语句来验证：

    select `NamespaceId`, `Key`, `Value`, `Comment` from ApolloConfigDB.Item;

|  | NamespaceId | Key | Value | Comment |
| :--- | :--- | :--- | :--- | :--- |
| 1 | timeout | 100 | sample | timeout配置 |

### 1.2.2.2.配置数据库连接信息

Apollo服务端需要知道如何连接到你前面创建的数据库，所以需要编辑demo.sh，修改ApolloPortalDB和ApolloConfigDB相关的数据库连接串信息。

```
# apollo config db info
apollo_config_db_url=jdbc:mysql://localhost:3306/ApolloConfigDB?characterEncoding=utf8
apollo_config_db_username=用户名
apollo_config_db_password=密码（如果没有密码，留空即可）

# apollo portal db info
apollo_portal_db_url=jdbc:mysql://localhost:3306/ApolloPortalDB?characterEncoding=utf8
apollo_portal_db_username=用户名
apollo_portal_db_password=密码（如果没有密码，留空即可）
```

### 1.2.3.启动Apollo配置中心

#### 1.2.3.1.确保端口未被占用

Quick Start脚本会在本地启动3个服务，分别使用8070, 8080, 8090端口，请确保这3个端口当前没有被使用。例如，在Linux/Mac下，可以通过如下命令检查：

```
lsof -i:8080
```

在windows下，可以通过如下命令检查：

```
netstat -aon|findstr "8080"
```

#### 1.2.3.2.执行启动脚本

在Quick Start目录下执行如下命令：

```
./demo.sh start
```

当看到如下输出后，就说明启动成功了！

```
==== starting service ====
Service logging file is ./service/apollo-service.log
Started [10768]
Waiting for config service startup.......
Config service started. You may visit http://localhost:8080 for service status now!
Waiting for admin service startup....
Admin service started
==== starting portal ====
Portal logging file is ./portal/apollo-portal.log
Started [10846]
Waiting for portal startup......
Portal started. You can visit http://localhost:8070 now!
```

#### 1.2.3.3.异常排查

如果启动遇到了异常，可以分别查看service和portal目录下的log文件排查问题。

注： 在启动apollo-configservice的过程中会在日志中输出eureka注册失败的信息，如com.sun.jersey.api.client.ClientHandlerException: java.net.ConnectException: Connection refused。需要注意的是，这个是预期的情况，因为apollo-configservice需要向Meta Server（它自己）注册服务，但是因为在启动过程中，自己还没起来，所以会报这个错。后面会进行重试的动作，所以等自己服务起来后就会注册正常了。

### 1.2.4.使用Apollo配置中心

#### 1.2.4.1.查看样例配置

1.打开[http://localhost:8070](http://localhost:8070)

![](/static/image/apollo-login.png)

Quick Start集成了Spring Security简单认证，更多信息可以参考

[Portal 实现用户登录功能](https://github.com/ctripcorp/apollo/wiki/Portal-实现用户登录功能)

2.输入用户名apollo，密码admin登录  
![](/static/image/apollo-sample-home.png)配置中心中包含一个默认的项目`SampleApp`

3.点击SampleApp进入配置界面，可以看到当前有一个配置timeout=100  
![](/static/image/sample-app-config.png)

如果提示系统出错，请重试或联系系统负责人，请稍后几秒钟重试一下，因为通过Eureka注册的服务有一个刷新的延时。

#### 1.2.4.2.新增项目配置

我们的客户端使用apollo需要新增相关的项目配置。  
1.点击新建项目

* 应用ID：这个ID是应用的唯一标识
* 应用名称：应用的名称，会展示在配置中心的首页上
  点击提交，创建完成
  2.新增配置信息
  点击新增配置，填写配置信息
  点击提交，此时配置还未生效。
  3.发布配置
  点击发布，配置立刻生效
  4.回滚
  如果配置做了修改之后，发现配置更改错误，这个时候可以使用回滚功能，回到上一个配置

## 1.3.Apollo配置中心客户端

我们客户端基于Spring Boot 2.0搭建，开发工具是InteIIij IDEA。新建一个项目，项目名称为apollo-client

### 1.3.1.客户端搭建

#### 1.3.1.1.添加Apollo客户端依赖

目前最新版本为1.1.1

```
<dependency>
     <groupId>com.ctrip.framework.apollo</groupId>
     <artifactId>apollo-client</artifactId>
     <version>1.1.1</version>
</dependency>
```

#### 1.3.1.2.添加配置信息

```
# 应用ID(在Apollo服务端新增项目添加的应用ID)
app.id=testclient
# apollo-configservice地址
apollo.meta=http://127.0.0.1:8080
```

#### 1.3.1.3.开启Apollo客户端

在项目的启动类上添加@EnableApolloConfig注解

#### 1.3.1.4.新增一个测试接口

```
  @RequestMapping("/index")
  public String hello(){
    return "hello man";
  }
```

#### 1.3.1.5.启动服务测试

在Apollo配置中心中，我们对该项目有一条配置server.port = 9000,启动服务，访问[http://localhost:9000/index，返回hello](http://localhost:9000/index，返回hello) man。证明，客户端是从服务端获取的配置。

### 1.3.2.客户端用法

在上一节，我们简单的搭建了客户端，成功的使用服务端配置。Apollo为我们提供的使用方式有很多种，下面只介绍Spring Boot 2.0环境下的使用方式。

#### 1.3.2.1.Spring Placeholder的使用

Spring应用通常会使用Placeholder来注入配置，使用的格式形如`${someKey:someDefaultValue}`，如  
`${timeout:100}`。冒号前面的是key，冒号后面的是默认值（建议在实际使用时尽量给出默认值，以免由于key没有定义导致运行时错误）。Apollo从v0.10.0开始的版本支持placeholder在运行时自动更新。如果需要关闭placeholder在运行时自动更新功能，可以通过以下两种方式关闭：

1.通过设置`System Property apollo.autoUpdateInjectedSpringProperties`，如启动时传入  
`-Dapollo.autoUpdateInjectedSpringProperties=false`  
2.通过设置META-INF/app.properties中的`apollo.autoUpdateInjectedSpringProperties=false`

#### 1.3.2.2.Java Config使用方式

1.新建配置类JavaConfigBean如下：

```
/**
 * Java Config方式
 *
 * @author simon
 * @create 2018-11-02 15:00
 **/
@Configuration
public class JavaConfigBean {
  @Value("${timeout:20}")
  private int timeout;

  public int getTimeout() {
    return timeout;
  }
}
```

2.新增访问端点

```
  //1.Java Config方式
  @Autowired
  JavaConfigBean javaConfigBean;

  @RequestMapping("/index1")
  public String hello1(){
    return javaConfigBean.getTimeout()+"";
  }
```

3.测试  
浏览器访问`http://127.0.0.1:8080/index1`，正确返回配置的值

#### 1.3.2.3.ConfigurationProperties使用方式

Spring Boot提供了`@ConfigurationProperties`把配置注入到bean对象中。Apollo也支持这种方式，下面的例子会把

`redis.cache.expireSeconds`和`redis.cache.commandTimeout`分别注入到`SampleRedisConfig`的`expireSeconds`和`commandTimeout`

字段中。

1.新增配置类`SampleRedisConfig`如下：

