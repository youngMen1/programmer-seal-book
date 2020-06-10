# 1.SpringBoot的main方法启动类SpringApplication需要在项目根目录

## 1.1.概述

* 使用SpringBoot的应用是需要将应用代码编译打包成jar包，然后基于main方法的方式来独立启动这个应用，使得该应用作为一个独立进程运行。这是跟传统的将项目打包成war包，然后部署到tomcat服务器去运行的一个区别。

* 而在应用当中，这个包含main方法的启动类需要放在项目的根目录，与所有包平级，一般在main方法内部通过执行SpringApplication.run方法来启动应用。启动类自身是一个基于注解的配置类，一般使用@SpringBootApplication注解，而这个注解由三个注解组成，分别是：@SpringBootConfiguration，@ComonentScan，@EnableAutoConfiguration。所以也可以单独使用这三个注解。

* 一个典型的SpringBoot项目结构如下：

![](/static/image/20190607223852164.png)

## 1.2.注解分析



