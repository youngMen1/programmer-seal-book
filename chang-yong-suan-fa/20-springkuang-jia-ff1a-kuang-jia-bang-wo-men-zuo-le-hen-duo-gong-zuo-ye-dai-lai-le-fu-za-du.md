# 20 | Spring框架：框架帮我们做了很多工作也带来了复杂度

你好，我是朱晔。今天，我们聊聊 Spring 框架给业务代码带来的复杂度，以及与之相关的坑。

在上一讲，通过 AOP 实现统一的监控组件的案例，我们看到了 IoC 和 AOP 配合使用的威力：当对象由 Spring 容器管理成为 Bean 之后，我们不但可以通过容器管理配置 Bean 的属性，还可以方便地对感兴趣的方法做 AOP。

不过，前提是对象必须是 Bean。你可能会觉得这个结论很明显，也很容易理解啊。但就和上一讲提到的 Bean 默认是单例一样，理解起来简单，实践的时候却非常容易踩坑。其中原因，一方面是，理解 Spring 的体系结构和使用方式有一定曲线；另一方面是，Spring 多年发展堆积起来的内部结构非常复杂，这也是更重要的原因。

在我看来，Spring 框架内部的复杂度主要表现为三点：

第一，Spring 框架借助 IoC 和 AOP 的功能，实现了修改、拦截 Bean 的定义和实例的灵活性，因此真正执行的代码流程并不是串行的。

第二，Spring Boot 根据当前依赖情况实现了自动配置，虽然省去了手动配置的麻烦，但也因此多了一些黑盒、提升了复杂度。

第三，Spring Cloud 模块多版本也多，Spring Boot 1.x 和 2.x 的区别也很大。如果要对 Spring Cloud 或 Spring Boot 进行二次开发的话，考虑兼容性的成本会很高。

今天，我们就通过配置 AOP 切入 Spring Cloud Feign 组件失败、Spring Boot 程序的文件配置被覆盖这两个案例，感受一下 Spring 的复杂度。我希望这一讲的内容，能帮助你面对 Spring 这个复杂框架出现的问题时，可以非常自信地找到解决方案。

## Feign AOP 切不到的诡异案例

我曾遇到过这么一个案例：使用 Spring Cloud 做微服务调用，为方便统一处理 Feign，想到了用 AOP 实现，即使用 within 指示器匹配 feign.Client 接口的实现进行 AOP 切入。

代码如下，通过 @Before 注解在执行方法前打印日志，并在代码中定义了一个标记了 @FeignClient 注解的 Client 类，让其成为一个 Feign 接口：


```

//测试Feign
@FeignClient(name = "client")
public interface Client {
    @GetMapping("/feignaop/server")
    String api();
}

//AOP切入feign.Client的实现
@Aspect
@Slf4j
@Component
public class WrongAspect {
    @Before("within(feign.Client+)")
    public void before(JoinPoint pjp) {
        log.info("within(feign.Client+) pjp {}, args:{}", pjp, pjp.getArgs());
    }
}

//配置扫描Feign
@Configuration
@EnableFeignClients(basePackages = "org.geekbang.time.commonmistakes.spring.demo4.feign")
public class Config {
}

```

通过 Feign 调用服务后可以看到日志中有输出，的确实现了 feign.Client 的切入，切入的是 execute 方法：


```

[15:48:32.850] [http-nio-45678-exec-1] [INFO ] [o.g.t.c.spring.demo4.WrongAspect        :20  ] - within(feign.Client+) pjp execution(Response feign.Client.execute(Request,Options)), args:[GET http://client/feignaop/server HTTP/1.1

Binary data, feign.Request$Options@5c16561a]
```

一开始这个项目使用的是客户端的负载均衡，也就是让 Ribbon 来做负载均衡，代码没啥问题。后来因为后端服务通过 Nginx 实现服务端负载均衡，所以开发同学把 @FeignClient 的配置设置了 URL 属性，直接通过一个固定 URL 调用后端服务：


```

@FeignClient(name = "anotherClient",url = "http://localhost:45678")
public interface ClientWithUrl {
    @GetMapping("/feignaop/server")
    String api();
}
```

但这样配置后，之前的 AOP 切面竟然失效了，也就是 within(feign.Client+) 无法切入 ClientWithUrl 的调用了。

为了还原这个场景，我写了一段代码，定义两个方法分别通过 Client 和 ClientWithUrl 这两个 Feign 进行接口调用：



```

@Autowired
private Client client;

@Autowired
private ClientWithUrl clientWithUrl;

@GetMapping("client")
public String client() {
    return client.api();
}

@GetMapping("clientWithUrl")
public String clientWithUrl() {
    return clientWithUrl.api();
}
```

可以看到，调用 Client 后 AOP 有日志输出，调用 ClientWithUrl 后却没有：


```

[15:50:32.850] [http-nio-45678-exec-1] [INFO ] [o.g.t.c.spring.demo4.WrongAspect        :20  ] - within(feign.Client+) pjp execution(Response feign.Client.execute(Request,Options)), args:[GET http://client/feignaop/server HTTP/1.1

Binary data, feign.Request$Options@5c16561
```
这就很费解了。难道为 Feign 指定了 URL，其实现就不是 feign.Clinet 了吗？


要明白原因，我们需要分析一下 FeignClient 的创建过程，也就是分析 FeignClientFactoryBean 类的 getTarget 方法。源码第 4 行有一个 if 判断，当 URL 没有内容也就是为空或者不配置时调用 loadBalance 方法，在其内部通过 FeignContext 从容器获取 feign.Client 的实例：


```

<T> T getTarget() {
  FeignContext context = this.applicationContext.getBean(FeignContext.class);
  Feign.Builder builder = feign(context);
  if (!StringUtils.hasText(this.url)) {
    ...
    return (T) loadBalance(builder, context,
        new HardCodedTarget<>(this.type, this.name, this.url));
  }
  ...
  String url = this.url + cleanPath();
  Client client = getOptional(context, Client.class);
  if (client != null) {
    if (client instanceof LoadBalancerFeignClient) {
      // not load balancing because we have a url,
      // but ribbon is on the classpath, so unwrap
      client = ((LoadBalancerFeignClient) client).getDelegate();
    }
    builder.client(client);
  }
  ...
}
protected <T> T loadBalance(Feign.Builder builder, FeignContext context,
    HardCodedTarget<T> target) {
  Client client = getOptional(context, Client.class);
  if (client != null) {
    builder.client(client);
    Targeter targeter = get(context, Targeter.class);
    return targeter.target(this, builder, context, target);
  }
...
}
protected <T> T getOptional(FeignContext context, Class<T> type) {
  return context.getInstance(this.contextId, type);
}

```

调试一下可以看到，client 是 LoadBalanceFeignClient，已经是经过代理增强的，明显是一个 Bean：

0510e28cd764aaf7f1b4b4ca03049ffd.png

所以，没有指定 URL 的 @FeignClient 对应的 LoadBalanceFeignClient，是可以通过 feign.Client 切入的。

在我们上面贴出来的源码的 16 行可以看到，当 URL 不为空的时候，client 设置为了 LoadBalanceFeignClient 的 delegate 属性。其原因注释中有提到，因为有了 URL 就不需要客户端负载均衡了，但因为 Ribbon 在 classpath 中，所以需要从 LoadBalanceFeignClient 提取出真正的 Client。断点调试下可以看到，这时 client 是一个 ApacheHttpClient：

1b872a900be7327f74bc09bde4c54230.png

那么，这个 ApacheHttpClient 是从哪里来的呢？这里，我教你一个小技巧：如果你希望知道一个类是怎样调用栈初始化的，可以在构造方法中设置一个断点进行调试。这样，你就可以在 IDE 的栈窗口看到整个方法调用栈，然后点击每一个栈帧看到整个过程。