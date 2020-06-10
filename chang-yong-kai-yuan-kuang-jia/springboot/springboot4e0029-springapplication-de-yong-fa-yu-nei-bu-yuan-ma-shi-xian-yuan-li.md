# 1.SpringApplication的用法与内部源码实现原理

### 1.1.概述

* 在基于SpringBoot的web应用中，通常使用一个带有main方法的类，通过命令行执行main方法来启动整个应用。而在main方法中是使用SpringApplication.run这个静态方法或者创建SpringApplication对象，执行成员方法run，以该main方法所在的类作为参数的方式启动的。

* main方法所在的类是一个基于Spring的注解，如@Configuration，@ComponentScan等，的配置类。

# 2.源码实现分析

### 2.1.Spring Boot 的入口类

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
 // 第一个参数 primarySource：加载的主要资源类
 // 第二个参数 args：传递给应用的应用参数
 public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
        return run(new Class[]{primarySource}, args);
    }

    public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
        return (new SpringApplication(primarySources)).run(args);
    }
```

第一个参数 `primarySource`：加载的主要资源类

第二个参数 `args`：传递给应用的应用参数

先用主要资源类来实例化一个 `SpringApplication` 对象，再调用这个对象的 `run` 方法，所以我们分两步来分析这个启动源码。

### 2.2.SpringApplication 的实例化过程

```
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
        return (new SpringApplication(primarySources)).run(args);
    }
```

接着上面的`SpringApplication`构造方法进入以下源码：

```
public SpringApplication(Class... primarySources) {
        this((ResourceLoader)null, primarySources);
    }

public SpringApplication(ResourceLoader resourceLoader, Class... primarySources) {
        this.sources = new LinkedHashSet();
        this.bannerMode = Mode.CONSOLE;
        this.logStartupInfo = true;
        this.addCommandLineProperties = true;
        this.addConversionService = true;
        this.headless = true;
        this.registerShutdownHook = true;
        this.additionalProfiles = new HashSet();
        this.isCustomEnvironment = false;
        // 1、资源初始化，资源加载器为 null
        this.resourceLoader = resourceLoader;

        // 2、断言主要加载资源类不能为 null，否则报错
        Assert.notNull(primarySources, "PrimarySources must not be null");

        // 3、初始化主要加载资源类集合并去重
        this.primarySources = new LinkedHashSet(Arrays.asList(primarySources));

        // 4、推断当前 WEB 应用类型
        this.webApplicationType = WebApplicationType.deduceFromClasspath();

        // 5、设置应用上下文初始化器
        this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class));

        // 6、设置监听器
        this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));

        // 7、推断主入口应用类
        this.mainApplicationClass = this.deduceMainApplicationClass();
    }
```

**可知这个构造器类的初始化包括以下 7 个过程。**

## 2.3.SpringApplication**构造器类初始化7个过程具体分析**

### **1. 资源初始化资源加载器为 null**

```
this.resourceLoader = resourceLoader;
```

### **2. 断言主要加载资源类不能为 null，否则报错**

```
Assert.notNull(primarySources, "PrimarySources must not be null");
```

### **3. 初始化主要加载资源类集合并去重**

```
this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
```

### **4. 推断当前 WEB 应用类型**

```
this.webApplicationType = deduceWebApplicationType();
```

**来看下**`deduceWebApplicationType`**方法和相关的源码：**

```
static WebApplicationType deduceFromClasspath() {
    if (ClassUtils.isPresent("org.springframework.web.reactive.DispatcherHandler", (ClassLoader)null) && !ClassUtils.isPresent("org.springframework.web.servlet.DispatcherServlet", (ClassLoader)null) && !ClassUtils.isPresent("org.glassfish.jersey.servlet.ServletContainer", (ClassLoader)null)) {
        return REACTIVE;
    } else {
        String[] var0 = SERVLET_INDICATOR_CLASSES;
        int var1 = var0.length;

        for(int var2 = 0; var2 < var1; ++var2) {
            String className = var0[var2];
            if (!ClassUtils.isPresent(className, (ClassLoader)null)) {
                return NONE;
            }
        }

        return SERVLET;
    }
}

public enum WebApplicationType {

/**
* 非 WEB 项目
*/

NONE,


/**
* SERVLET WEB 项目
*/

SERVLET,


/**
* 响应式 WEB 项目
*/

REACTIVE
}
```

这个就是根据类路径下是否有对应项目类型的类推断出不同的应用类型。

### 5.设置应用上线文初始化器

```

```

### 6.设置监听器

### 7.推断主入口应用类



