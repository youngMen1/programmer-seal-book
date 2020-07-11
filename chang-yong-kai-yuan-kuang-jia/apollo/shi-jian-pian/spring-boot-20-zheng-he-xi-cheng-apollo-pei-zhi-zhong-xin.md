# 1.Spring Boot 2.0 整合携程Apollo配置中心

Apollo（阿波罗）是携程框架部门研发的分布式配置中心，能够集中化管理应用不同环境、不同集群的配置，配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性，适用于微服务配置管理场景。

服务端基于Spring Boot和Spring Cloud开发，打包后可以直接运行，不需要额外安装Tomcat等应用容器。

Java客户端不依赖任何框架，能够运行于所有Java运行时环境，同时对Spring/Spring Boot环境也有较好的支持。

.Net客户端不依赖任何框架，能够运行于所有.Net运行时环境。

如果想要深入了解，可以到github上参见

[Apollo配置中心](https://github.com/ctripcorp/apollo#screenshots)

，官网的介绍很详细。本章主要讲述Spring Boot 2.0 整合Apollo配置中心。

