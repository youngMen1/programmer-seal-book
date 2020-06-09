# 1.一般问题

## 1.1. 不同版本的 Spring Framework 有哪些主要功能？

| Version | Feature |
| :--- | :--- |
| Spring 2.5 | 发布于 2007 年。这是第一个支持注解的版本。 |
| Spring 3.0 | 发布于 2009 年。它完全利用了 Java5 中的改进，并为 JEE6 提供了支持。 |
| Spring 4.0 | 发布于 2013 年。这是第一个完全支持 JAVA8 的版本。 |
| Spring5.0 | 最大特点之一是响应式编程（Reactive Programming） |

## 1.2. 什么是 Spring Framework？

* Spring 是一个开源应用框架，旨在降低应用程序开发的复杂度。
* 它是轻量级、松散耦合的。
* 它具有分层体系结构，允许用户选择组件，同时还为 J2EE 应用程序开发提供了一个有凝聚力的框架。
* 它可以集成其他框架，如 Structs、Hibernate、EJB 等，所以又称为框架的框架。

## 1.3. 列举 Spring Framework 的优点。

* 由于 Spring Frameworks 的分层架构，用户可以自由选择自己需要的组件。
* Spring Framework 支持 POJO\(Plain Old Java Object\) 编程，从而具备持续集成和可测试性。
* 由于依赖注入和控制反转，JDBC 得以简化。
* 它是开源免费的。

## 1.4. Spring Framework 有哪些不同的功能？

* 轻量级 - Spring 在代码量和透明度方面都很轻便。
* IOC - 控制反转
* AOP - 面向切面编程可以将应用业务逻辑和系统服务分离，以实现高内聚。
* 容器 - Spring 负责创建和管理对象（Bean）的生命周期和配置。
* MVC - 对 web 应用提供了高度可配置性，其他框架的集成也十分方便。
* 事务管理 - 提供了用于事务管理的通用抽象层。Spring 的事务支持也可用于容器较少的环境。
* JDBC 异常 - Spring 的 JDBC 抽象层提供了一个异常层次结构，简化了错误处理策略。

## 1.5. Spring Framework 中有多少个模块，它们分别是什么？

![](/static/image/20190804193644516.png)

1.5.1.Spring 核心容器（Core Container） – 该层基本上是 Spring Framework 的核心。它包含以下模块：

* Spring Bean
* Spring Core
* Spring Context
* SpEL \(Spring Expression Language\)

数据访问/集成（Data Access/Integration） – 该层提供与数据库交互的支持。它包含以下模块：

* JDBC \(Java DataBase Connectivity\)
* ORM \(Object Relational Mapping\)
* OXM \(Object XML Mappers\)
* JMS \(Java Messaging Service\)
* Transaction

Web – 该层提供了创建 Web 应用程序的支持。它包含以下模块：

* Web – Socket
* Web
* Web – Servlet
* Web – Portlet

AOP – 该层支持面向切面编程

Instrumentation – 该层为类检测和类加载器实现提供支持。

Test – 该层为使用 JUnit 和 TestNG 进行测试提供支持。

Messaging – 该模块为 STOMP 提供支持。它还支持注解编程模型，该模型用于从 WebSocket 客户端路由和处理 STOMP 消息。

Aspects – 该模块为与 AspectJ 的集成提供支持。

## 1.6. 什么是 Spring 配置文件？

Spring 配置文件是 XML 文件。该文件主要包含类信息。它描述了这些类是如何配置以及相互引入的。但是，XML 配置文件冗长且更加干净。如果没有正确规划和编写，那么在大项目中管理变得非常困难。

## 1.7. Spring 应用程序有哪些不同组件？

Spring 应用一般有以下组件：

* 接口 - 定义功能。
* Bean 类 - 它包含属性，setter 和 getter 方法，函数等。
* Spring 面向切面编程（AOP） - 提供面向切面编程的功能。
* Bean 配置文件 - 包含类的信息以及如何配置它们。
* 用户程序 - 它使用接口。

## 1.8. 使用 Spring 有哪些方式？

使用 Spring 有以下方式：

* 作为一个成熟的 Spring Web 应用程序。
* 作为第三方 Web 框架，使用 Spring Frameworks 中间层。
* 用于远程使用。
* 作为企业级 Java Bean，它可以包装现有的 POJO（Plain Old Java Objects）。

# 2.依赖注入（Ioc）

## 2.1. 什么是 Spring IOC 容器？

Spring 框架的核心是 Spring 容器。容器创建对象，将它们装配在一起，配置它们并管理它们的完整生命周期。Spring 容器使用依赖注入来管理组成应用程序的组件。容器通过读取提供的配置元数据来接收对象进行实例化，配置和组装的指令。该元数据可以通过 XML，Java 注解或 Java 代码提供。

20190804193749449.png

## 2.2. 什么是依赖注入？

在依赖注入中，您不必创建对象，但必须描述如何创建它们。您不是直接在代码中将组件和服务连接在一起，而是描述配置文件中哪些组件需要哪些服务。由 IoC 容器将它们装配在一起。

## 2.3. 可以通过多少种方式完成依赖注入？

通常，依赖注入可以通过三种方式完成，即：

* 构造函数注入
* setter 注入
* 接口注入
* 在 Spring Framework 中，仅使用构造函数和 setter 注入。

## 2.4. 区分构造函数注入和 setter 注入。

| 构造函数注入 | setter 注入 |
| :--- | :--- |
| 没有部分注入 | 有部分注入 |
| 不会覆盖 setter 属性 | 会覆盖 setter 属性 |
| 任意修改都会创建一个新实例 | 任意修改不会创建一个新实例 |
| 适用于设置很多属性 | 适用于设置少量属性 |

## 2.5. spring 中有多少种 IOC 容器？

* BeanFactory - BeanFactory 就像一个包含 bean 集合的工厂类。它会在客户端要求时实例化 bean。

* ApplicationContext - ApplicationContext 接口扩展了 BeanFactory 接口。它在 BeanFactory 基础上提供了一些额外的功能。

## 2.6. 区分 BeanFactory 和 ApplicationContext。

| BeanFactory |
| :--- |


|  | ApplicationContext |
| :--- | :--- |
| 它使用懒加载 | 它使用即时加载 |
| 它使用语法显式提供资源对象 | 它自己创建和管理资源对象 |
| 不支持国际化 | 支持国际化 |
| 不支持基于依赖的注解 | 支持基于依赖的注解 |

# 3.Beans

## 4. 注解

## 5. 数据访问

## 6. AOP

## 7. MVC

# 8.参考

Spring 面试问题 TOP 50：

[https://blog.csdn.net/forezp/article/details/84927048](https://blog.csdn.net/forezp/article/details/84927048)

Spring Framework 5.0 新特性：

[https://www.jianshu.com/p/cebc3cf0bec0](https://www.jianshu.com/p/cebc3cf0bec0)

