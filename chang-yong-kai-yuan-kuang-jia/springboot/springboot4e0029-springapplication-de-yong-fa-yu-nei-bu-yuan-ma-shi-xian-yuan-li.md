# 1.SpringBoot启动全过程源码分析SpringApplication的用法原理

### 1.1.概述

* 在基于SpringBoot的web应用中，通常使用一个带有main方法的类，通过命令行执行main方法来启动整个应用。而在main方法中是使用SpringApplication.run这个静态方法或者创建SpringApplication对象，执行成员方法run，以该main方法所在的类作为参数的方式启动的。

* main方法所在的类是一个基于Spring的注解，如@Configuration，@ComponentScan等，的配置类。

# 2.源码实现分析

![](/static/image/retlrejtletjmeltmerktgnelrtgner.webp)

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
setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
```

`ApplicationContextInitializer`**的作用是什么？源码如下。**

```
public interface ApplicationContextInitializer<C extends ConfigurableApplicationContext> {

/**

* Initialize the given application context.

* @param applicationContext the application to configure

*/

void initialize(C applicationContext);
}
```

用来初始化指定的 Spring 应用上下文，如**注册属性资源、激活 Profiles** 等。

来看下 `setInitializers` 方法源码，其实就是初始化一个 `ApplicationContextInitializer` 应用上下文初始化器实例的集合。

```
/**

* Sets the {@link ApplicationContextInitializer} that will be applied to the Spring

* {@link ApplicationContext}.

* @param initializers the initializers to set

*/
public void setInitializers(Collection<? extends ApplicationContextInitializer<?>> initializers) {

this.initializers = new ArrayList<>(initializers);

}
```

再来看下这个初始化`getSpringFactoriesInstances`方法和相关的源码：

```
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type) {

 return getSpringFactoriesInstances(type, new Class<?>[] {});
 }


private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) 
 { 

  // 1.获取当前线程上下文类加载器
  ClassLoader classLoader = getClassLoader();

  // Use names and ensure unique to protect against duplicates
  // 2.获取 ApplicationContextInitializer 的实例名称集合并去重
  Set<String> names = new LinkedHashSet<>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));

  // 3.根据以上类路径创建初始化器实例列表
  List<T> instances = createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);

  // 4.初始化器实例列表排序
   AnnotationAwareOrderComparator.sort(instances);

   return instances;

}
```

#### 设置应用上下文初始化器可分为以下 5 个步骤。

##### 1.**获取当前线程上下文类加载器**

```
ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
```

##### 2.**获取 **`ApplicationContextInitializer`**的实例名称集合并去重**

```
Set<String> names = new LinkedHashSet<>(
               SpringFactoriesLoader.loadFactoryNames(type, classLoader));
```

`loadFactoryNames`**方法相关的源码如下：**

```
public static List<String> loadFactoryNames(Class<?> factoryClass, @Nullable ClassLoader classLoader) {

String factoryClassName = factoryClass.getName();
   return loadSpringFactories(classLoader).getOrDefault(factoryClassName, Collections.emptyList());
}
public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";
private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
   MultiValueMap<String, String> result = cache.get(classLoader);
   if (result != null) {
       return result;
   }
   try {
       Enumeration<URL> urls = (classLoader != null ?
               classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
               ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
       result = new LinkedMultiValueMap<>();
       while (urls.hasMoreElements()) {
           URL url = urls.nextElement();
           UrlResource resource = new UrlResource(url);
           Properties properties = PropertiesLoaderUtils.loadProperties(resource);
           for (Map.Entry<?, ?> entry : properties.entrySet()) {
               List<String> factoryClassNames = Arrays.asList(
                       StringUtils.commaDelimitedListToStringArray((String) entry.getValue()));
               result.addAll((String) entry.getKey(), factoryClassNames);
           }
       }
       cache.put(classLoader, result);
       return result;
   }
   catch (IOException ex) {
       throw new IllegalArgumentException("Unable to load factories from location [" +
               FACTORIES_RESOURCE_LOCATION + "]", ex);
   }
}
```

根据类路径下的`META-INF/spring.factories`文件解析并获取`ApplicationContextInitializer`接口的所有配置的类路径名称。

`spring-boot-autoconfigure-2.0.3.RELEASE.jar!/META-INF/spring.factories`的初始化器相关配置内容如下：

```
# Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,\
org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener
```

##### 3.**根据以上类路径创建初始化器实例列表**

```
List<T> instances = createSpringFactoriesInstances(type, parameterTypes,
               classLoader, args, names);
private <T> List<T> createSpringFactoriesInstances(Class<T> type,
       Class<?>[] parameterTypes, ClassLoader classLoader, Object[] args,
       Set<String> names) {
   List<T> instances = new ArrayList<>(names.size());
   for (String name : names) {
       try {
           Class<?> instanceClass = ClassUtils.forName(name, classLoader);
           Assert.isAssignable(type, instanceClass);
           Constructor<?> constructor = instanceClass
                   .getDeclaredConstructor(parameterTypes);
           T instance = (T) BeanUtils.instantiateClass(constructor, args);
           instances.add(instance);
       }
       catch (Throwable ex) {
           throw new IllegalArgumentException(
                   "Cannot instantiate " + type + " : " + name, ex);
       }
   }
   return instances;
}
```

##### 4.**初始化器实例列表排序**

```
AnnotationAwareOrderComparator.sort(instances);
```

##### 5.**返回初始化器实例列表**

```
return instances;
```

### 6.设置监听器

```
setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
```

`ApplicationListener`**的作用是什么？源码如下。**

```
@FunctionalInterface
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
   /**
    * Handle an application event.
    * @param event the event to respond to
    */
   void onApplicationEvent(E event);
}
```

看源码，这个接口继承了 JDK 的 `java.util.EventListener` 接口，实现了观察者模式，它一般用来定义感兴趣的事件类型，事件类型限定于 ApplicationEvent 的子类，这同样继承了 JDK 的 `java.util.EventObject` 接口。

设置监听器和设置初始化器调用的方法是一样的，只是传入的类型不一样，设置监听器的接口类型为： `getSpringFactoriesInstances`，对应的 `spring-boot-autoconfigure-2.0.3.RELEASE.jar!/META-INF/spring.factories` 文件配置内容请见下方。

```
# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.boot.autoconfigure.BackgroundPreinitializer
```

可以看出目前只有一个`BackgroundPreinitializer`监听器。

### 7.推断主入口应用类

```
this.mainApplicationClass = deduceMainApplicationClass();
private Class<?> deduceMainApplicationClass() {
   try {
       StackTraceElement[] stackTrace = new RuntimeException().getStackTrace();
       for (StackTraceElement stackTraceElement : stackTrace) {
           if ("main".equals(stackTraceElement.getMethodName())) {
               return Class.forName(stackTraceElement.getClassName());
           }
       }
   }
   catch (ClassNotFoundException ex) {
       // Swallow and continue
   }
   return null;
}
```

这个推断入口应用类的方式有点特别，通过构造一个运行时异常，再遍历异常栈中的方法名，获取方法名为 main 的栈帧，从来得到入口类的名字再返回该类。

