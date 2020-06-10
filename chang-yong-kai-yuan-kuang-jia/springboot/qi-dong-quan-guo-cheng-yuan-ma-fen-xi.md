# 1.SpringBoot启动全过程源码分析Run方法

## 1.1.SpringApplication 实例 run 方法运行过程

![](/static/image/sdfjsljflsmflisuowijvmskfms.webp)

上面分析了 SpringApplication 实例对象构造方法初始化过程，下面继续来看下这个 SpringApplication 对象的 run 方法的源码和运行流程。

```
/** * Run the Spring application, creating and refreshing a new * {@link ApplicationContext}. * @param args the application arguments (usually passed from a Java main method) * @return a running {@link ApplicationContext} */public ConfigurableApplicationContext run(String... args) {   StopWatch stopWatch = new StopWatch();   stopWatch.start();   ConfigurableApplicationContext context = null;   Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();   configureHeadlessProperty();   SpringApplicationRunListeners listeners = getRunListeners(args);   listeners.starting();   try {      ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);      ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);      configureIgnoreBeanInfo(environment);      Banner printedBanner = printBanner(environment);      context = createApplicationContext();      exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,            new Class[] { ConfigurableApplicationContext.class }, context);      prepareContext(context, environment, listeners, applicationArguments, printedBanner);      refreshContext(context);      afterRefresh(context, applicationArguments);      stopWatch.stop();      if (this.logStartupInfo) {         new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);      }      listeners.started(context);      callRunners(context, applicationArguments);   }   catch (Throwable ex) {      handleRunFailure(context, ex, exceptionReporters, listeners);      throw new IllegalStateException(ex);   }   try {      listeners.running(context);   }   catch (Throwable ex) {      handleRunFailure(context, ex, exceptionReporters, null);      throw new IllegalStateException(ex);   }   return context;}
```



