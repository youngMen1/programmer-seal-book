# 1.SpringBoot的main方法启动类SpringApplication需要在项目根目录

## 1.1.概述

* 使用SpringBoot的应用是需要将应用代码编译打包成jar包，然后基于main方法的方式来独立启动这个应用，使得该应用作为一个独立进程运行。这是跟传统的将项目打包成war包，然后部署到tomcat服务器去运行的一个区别。

* 而在应用当中，这个包含main方法的启动类需要放在项目的根目录，与所有包平级，一般在main方法内部通过执行SpringApplication.run方法来启动应用。启动类自身是一个基于注解的配置类，一般使用@SpringBootApplication注解，而这个@SpringBootApplication注解是由三个注解组成的，分别是：@SpringBootConfiguration，@ComonentScan，@EnableAutoConfiguration。所以也可以单独使用这三个注解。

* 一个典型的SpringBoot项目结构如下：

![](/static/image/20190607223852164.png)

## 1.2.注解分析

* @SpringBootConfiguration：配置类

继承于@Configuration，本身只是说明这是一个SpringBoot项目的配置类，功能与@Configuration一样，使得Spring容器知道需要跟处理@Configuration注解的类一样处理这个类。

* @ComponentScan：基于注解的类扫描

用于进行包扫描，检查类是否使用了@Controller，@Service等注解，有则获取这些类创建对应的bean对象注册到Spring容器；

* @EnableAutoConfiguration：SpringBoot的自动配置特性

该注解是SpringBoot引入的，用于自动配置，即基于项目配置pom.xml引入的SpringBoot的starter相关包和项目添加的配置类，判断是使用SpringBoot的starter包提供的配置类还是使用项目定义的配置类，如假如在pom.xml中引入了spring-boot-starter-data-redis包，则如果项目没有自定义RedisTemplate类实现，则SpringBoot会自动配置和注入一个RedisTemplate对象到Spring的IOC容器中。

* 工作过程为：扫描项目的所有包，检测项目中是否存在与SpringBoot自动添加的starter包对应功能组件类相同的类，或者实现了相同的接口或者继承了相同的父类的类，有则使用项目自身提供的该功能组件类实现，没有则使用SpringBoot自动添加的该功能组件类。SpringBoot的starter包自动添加的这些功能组件类通常是使用了@Configuration注解和@Conditional注解的，所以可以实现条件化注入。





