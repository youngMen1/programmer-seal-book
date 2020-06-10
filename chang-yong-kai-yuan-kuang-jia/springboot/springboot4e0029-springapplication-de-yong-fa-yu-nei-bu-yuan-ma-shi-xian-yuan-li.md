# 1.SpringApplication的用法与内部源码实现原理

### 1.1.概述

* 在基于SpringBoot的web应用中，通常使用一个带有main方法的类，通过命令行执行main方法来启动整个应用。而在main方法中是使用SpringApplication.run这个静态方法或者创建SpringApplication对象，执行成员方法run，以该main方法所在的类作为参数的方式启动的。

* main方法所在的类是一个基于Spring的注解，如@Configuration，@ComponentScan等，的配置类。

## 1.2.源码实现分析

### Spring Boot 的入口类

```
@SpringBootApplication
public class SpringBootBestPracticeApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBootBestPracticeApplication.class, args);
        }
}
```

做过 Spring Boot 项目的都知道，上面是 Spring Boot 最简单通用的入口类。入口类的要求是最顶层包下面第一个含有 main 方法的类，使用注解 `@SpringBootApplication` 来启用 Spring Boot 特性，使用 `SpringApplication.run` 方法来启动 Spring Boot 项目。

来看一下这个类的 `run` 方法调用关系源码：

```
 public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
        return run(new Class[]{primarySource}, args);
    }

    public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
        return (new SpringApplication(primarySources)).run(args);
    }
```



