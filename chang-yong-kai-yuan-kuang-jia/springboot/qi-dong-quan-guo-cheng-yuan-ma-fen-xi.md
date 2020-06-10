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
          Banner printedBanner = printBanner(environment);
          context = createApplicationContext();
          exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
                new Class[] { ConfigurableApplicationContext.class }, context);
          prepareContext(context, environment, listeners, applicationArguments, printedBanner);
          refreshContext(context);
          afterRefresh(context, applicationArguments);
          stopWatch.stop();
          if (this.logStartupInfo) {
             new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
          }
          listeners.started(context);
          callRunners(context, applicationArguments);
       }
       catch (Throwable ex) {
          handleRunFailure(context, ex, exceptionReporters, listeners);
          throw new IllegalStateException(ex);
       }

       try {
          listeners.running(context);
       }
       catch (Throwable ex) {
          handleRunFailure(context, ex, exceptionReporters, null);
          throw new IllegalStateException(ex);
       }
       return context;
    }



