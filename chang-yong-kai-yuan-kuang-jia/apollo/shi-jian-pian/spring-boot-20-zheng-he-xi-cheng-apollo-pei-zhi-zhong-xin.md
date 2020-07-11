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

官网为我们准备了 Quick Start 安装包。大家只需要下载到本地，就可以直接使用，免去了编译、打包过程。大家可以到

[github下载](https://github.com/nobodyiam/apollo-build-scripts)

，也可以通过

[百度云盘下载](https://pan.baidu.com/s/187W86LoeVuv3DMrOJhcg1A)

