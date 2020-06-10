# 1.SpringBoot启动全过程源码分析Run方法

## 1.1.SpringApplication 实例 run 方法运行过程

![](/static/image/sdfjsljflsmflisuowijvmskfms.webp)

上面分析了 SpringApplication 实例对象构造方法初始化过程，下面继续来看下这个 SpringApplication 对象的 run 方法的源码和运行流程。

    /**
     * Run the Spring application, creating and refreshing a new
     * {@link ApplicationContext}.
     * @param args the application arguments (usually passed from a Java main method)
     * @return a running {@link ApplicationContext}
     */
    public ConfigurableApplicationContext run(String... args) {
       // 1、创建并启动计时监控类
       StopWatch stopWatch = new StopWatch();
       stopWatch.start();

       // 2、初始化应用上下文和异常报告集合
       ConfigurableApplicationContext context = null;
       Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();

       // 3、设置系统属性 `java.awt.headless` 的值，默认值为：true
       configureHeadlessProperty();

       // 4、创建所有 Spring 运行监听器并发布应用启动事件
       SpringApplicationRunListeners listeners = getRunListeners(args);
       listeners.starting();

       try {
          // 5、初始化默认应用参数类
          ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);

          // 6、根据运行监听器和应用参数来准备 Spring 环境
          ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
          configureIgnoreBeanInfo(environment);

          // 7、创建 Banner 打印类
          Banner printedBanner = printBanner(environment);

          // 8、创建应用上下文
          context = createApplicationContext();

          // 9、准备异常报告器
          exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
                new Class[] { ConfigurableApplicationContext.class }, context);

          // 10、准备应用上下文
          prepareContext(context, environment, listeners, applicationArguments, printedBanner);

          // 11、刷新应用上下文
          refreshContext(context);

          // 12、应用上下文刷新后置处理
          afterRefresh(context, applicationArguments);

          // 13、停止计时监控类
          stopWatch.stop();

           // 14、输出日志记录执行主类名、时间信息
          if (this.logStartupInfo) {
             new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
          }

          // 15、发布应用上下文启动完成事件
          listeners.started(context);

          // 16、执行所有 Runner 运行器
          callRunners(context, applicationArguments);
       }
       catch (Throwable ex) {
          handleRunFailure(context, ex, exceptionReporters, listeners);
          throw new IllegalStateException(ex);
       }

       try {
          // 17、发布应用上下文就绪事件
          listeners.running(context);
       }
       catch (Throwable ex) {
          handleRunFailure(context, ex, exceptionReporters, null);
          throw new IllegalStateException(ex);
       }
       // 18、返回应用上下文
       return context;
    }

所以，我们可以按以下几步来分解 run 方法的启动过程。

# 2.源码实现分析

## 1.创建并启动计时监控类

```
StopWatch stopWatch = new StopWatch();
stopWatch.start();
```

来看下这个计时监控类 StopWatch 的相关源码：

```
public void start() throws IllegalStateException {
    start("");
}

public void start(String taskName) throws IllegalStateException {
    if (this.currentTaskName != null) {
        throw new IllegalStateException("Can't start StopWatch: it's already running");
    }
    this.currentTaskName = taskName;
    this.startTimeMillis = System.currentTimeMillis();
}
```

首先记录了当前任务的名称，默认为空字符串，然后记录当前 Spring Boot 应用启动的开始时间。

## 2.初始化应用上下文和异常报告集合

```
ConfigurableApplicationContext context = null;
Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
```

## 3.设置系统属性 \`java.awt.headless\` 的值

```
configureHeadlessProperty();
```

设置该默认值为：true，Java.awt.headless = true 有什么作用？

对于一个 Java 服务器来说经常要处理一些图形元素，例如地图的创建或者图形和图表等。这些API基本上总是需要运行一个X-server以便能使用AWT（Abstract Window Toolkit，抽象窗口工具集）。然而运行一个不必要的 X-server 并不是一种好的管理方式。有时你甚至不能运行 X-server,因此最好的方案是运行 headless 服务器，来进行简单的图像处理。

```
参考：www.cnblogs.com/princessd8251/p/4000016.html
```

## 4.创建所有 Spring 运行监听器并发布应用启动事件





