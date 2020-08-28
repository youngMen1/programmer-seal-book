# 19 | Spring框架：IoC和AOP是扩展的核心

熟悉 Java 的同学都知道，Spring 的家族庞大，常用的模块就有 Spring Data、Spring Security、Spring Boot、Spring Cloud 等。其实呢，Spring 体系虽然庞大，但都是围绕 Spring Core 展开的，而 Spring Core 中最核心的就是 IoC（控制反转）和 AOP（面向切面编程）。

概括地说，IoC 和 AOP 的初衷是解耦和扩展。理解这两个核心技术，就可以让你的代码变得更灵活、可随时替换，以及业务组件间更解耦。在接下来的两讲中，我会与你深入剖析几个案例，带你绕过业务中通过 Spring 实现 IoC 和 AOP 相关的坑。

为了便于理解这两讲中的案例，我们先回顾下 IoC 和 AOP 的基础知识。

IoC，其实就是一种设计思想。使用 Spring 来实现 IoC，意味着将你设计好的对象交给 Spring 容器控制，而不是直接在对象内部控制。那，为什么要让容器来管理对象呢？或许你能想到的是，使用 IoC 方便、可以实现解耦。但在我看来，相比于这两个原因，更重要的是 IoC 带来了更多的可能性。

如果以容器为依托来管理所有的框架、业务对象，我们不仅可以无侵入地调整对象的关系，还可以无侵入地随时调整对象的属性，甚至是实现对象的替换。这就使得框架开发者在程序背后实现一些扩展不再是问题，带来的可能性是无限的。比如我们要监控的对象如果是 Bean，实现就会非常简单。所以，这套容器体系，不仅被 Spring Core 和 Spring Boot 大量依赖，还实现了一些外部框架和 Spring 的无缝整合。

AOP，体现了松耦合、高内聚的精髓，在切面集中实现横切关注点（缓存、权限、日志等），然后通过切点配置把代码注入合适的地方。切面、切点、增强、连接点，是 AOP 中非常重要的概念，也是我们这两讲会大量提及的。

为方便理解，我们把 Spring AOP 技术看作为蛋糕做奶油夹层的工序。如果我们希望找到一个合适的地方把奶油注入蛋糕胚子中，那应该如何指导工人完成操作呢？
c71f2ec73901f7bcaa8332f237dfeddb.png


* 首先，我们要提醒他，只能往蛋糕胚子里面加奶油，而不能上面或下面加奶油。这就是连接点（Join point），对于 Spring AOP 来说，连接点就是方法执行。

* 然后，我们要告诉他，在什么点切开蛋糕加奶油。比如，可以在蛋糕坯子中间加入一层奶油，在中间切一次；也可以在中间加两层奶油，在 1/3 和 2/3 的地方切两次。这就是切点（Pointcut），Spring AOP 中默认使用 AspectJ 查询表达式，通过在连接点运行查询表达式来匹配切入点。

* 接下来也是最重要的，我们要告诉他，切开蛋糕后要做什么，也就是加入奶油。这就是增强（Advice），也叫作通知，定义了切入切点后增强的方式，包括前、后、环绕等。Spring AOP 中，把增强定义为拦截器。

* 最后，我们要告诉他，找到蛋糕胚子中要加奶油的地方并加入奶油。为蛋糕做奶油夹层的操作，对 Spring AOP 来说就是切面（Aspect），也叫作方面。切面 = 切点 + 增强。

好了，理解了这几个核心概念，我们就可以继续分析案例了。

我要首先说明的是，Spring 相关问题的问题比较复杂，一方面是 Spring 提供的 IoC 和 AOP 本就灵活，另一方面 Spring Boot 的自动装配、Spring Cloud 复杂的模块会让问题排查变得更复杂。因此，今天这一讲，我会带你先打好基础，通过两个案例来重点聊聊 IoC 和 AOP；然后，我会在下一讲中与你分享 Spring 相关的坑。

## 单例的 Bean 如何注入 Prototype 的 Bean？

我们虽然知道 Spring 创建的 Bean 默认是单例的，但当 Bean 遇到继承的时候，可能会忽略这一点。为什么呢？忽略这一点又会造成什么影响呢？

接下来，我就和你分享一个由单例引起内存泄露的案例。

架构师一开始定义了这么一个 SayService 抽象类，其中维护了一个类型是 ArrayList 的字段 data，用于保存方法处理的中间数据。每次调用 say 方法都会往 data 加入新数据，可以认为 SayService 是有状态，如果 SayService 是单例的话必然会 OOM：



```

@Slf4j
public abstract class SayService {
    List<String> data = new ArrayList<>();

    public void say() {
        data.add(IntStream.rangeClosed(1, 1000000)
                .mapToObj(__ -> "a")
                .collect(Collectors.joining("")) + UUID.randomUUID().toString());
        log.info("I'm {} size:{}", this, data.size());
    }
}
```
但实际开发的时候，开发同学没有过多思考就把 SayHello 和 SayBye 类加上了 @Service 注解，让它们成为了 Bean，也没有考虑到父类是有状态的：


```

@Service
@Slf4j
public class SayHello extends SayService {
    @Override
    public void say() {
        super.say();
        log.info("hello");
    }
}

@Service
@Slf4j
public class SayBye extends SayService {
    @Override
    public void say() {
        super.say();
        log.info("bye");
    }
}
```

许多开发同学认为，@Service 注解的意义在于，能通过 @Autowired 注解让 Spring 自动注入对象，就比如可以直接使用注入的 List获取到 SayHello 和 SayBye，而没想过类的生命周期：


```

@Autowired
List<SayService> sayServiceList;

@GetMapping("test")
public void test() {
    log.info("====================");
    sayServiceList.forEach(SayService::say);
}
```


这一个点非常容易忽略。开发基类的架构师将基类设计为有状态的，但并不知道子类是怎么使用基类的；而开发子类的同学，没多想就直接标记了 @Service，让类成为了 Bean，通过 @Autowired 注解来注入这个服务。但这样设置后，有状态的基类就可能产生内存泄露或线程安全问题。

正确的方式是，**在为类标记上 @Service 注解把类型交由容器管理前，首先评估一下类是否有状态，然后为 Bean 设置合适的 Scope。**好在上线前，架构师发现了这个内存泄露问题，开发同学也做了修改，为 SayHello 和 SayBye 两个类都标记了 @Scope 注解，设置了 PROTOTYPE 的生命周期，也就是多例：



```

@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
```

但，上线后还是出现了内存泄漏，证明修改是无效的。

从日志可以看到，第一次调用和第二次调用的时候，SayBye 对象都是 4c0bfe9e，SayHello 也是一样的问题。从日志第 7 到 10 行还可以看到，第二次调用后 List 的元素个数变为了 2，说明父类 SayService 维护的 List 在不断增长，不断调用必然出现 OOM：


```

[15:01:09.349] [http-nio-45678-exec-1] [INFO ] [.s.d.BeanSingletonAndOrderController:22  ] - ====================
[15:01:09.401] [http-nio-45678-exec-1] [INFO ] [o.g.t.c.spring.demo1.SayService         :19  ] - I'm org.geekbang.time.commonmistakes.spring.demo1.SayBye@4c0bfe9e size:1
[15:01:09.402] [http-nio-45678-exec-1] [INFO ] [t.commonmistakes.spring.demo1.SayBye:16  ] - bye
[15:01:09.469] [http-nio-45678-exec-1] [INFO ] [o.g.t.c.spring.demo1.SayService         :19  ] - I'm org.geekbang.time.commonmistakes.spring.demo1.SayHello@490fbeaa size:1
[15:01:09.469] [http-nio-45678-exec-1] [INFO ] [o.g.t.c.spring.demo1.SayHello           :17  ] - hello
[15:01:15.167] [http-nio-45678-exec-2] [INFO ] [.s.d.BeanSingletonAndOrderController:22  ] - ====================
[15:01:15.197] [http-nio-45678-exec-2] [INFO ] [o.g.t.c.spring.demo1.SayService         :19  ] - I'm org.geekbang.time.commonmistakes.spring.demo1.SayBye@4c0bfe9e size:2
[15:01:15.198] [http-nio-45678-exec-2] [INFO ] [t.commonmistakes.spring.demo1.SayBye:16  ] - bye
[15:01:15.224] [http-nio-45678-exec-2] [INFO ] [o.g.t.c.spring.demo1.SayService         :19  ] - I'm org.geekbang.time.commonmistakes.spring.demo1.SayHello@490fbeaa size:2
[15:01:15.224] [http-nio-45678-exec-2] [INFO ] [o.g.t.c.spring.demo1.SayHello           :17  ] - hello
```
这就引出了单例的 Bean 如何注入 Prototype 的 Bean 这个问题。Controller 标记了 @RestController 注解，而 @RestController 注解 =@Controller 注解 +@ResponseBody 注解，又因为 @Controller 标记了 @Component 元注解，所以 @RestController 注解其实也是一个 Spring Bean：


```

//@RestController注解=@Controller注解+@ResponseBody注解@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller
@ResponseBody
public @interface RestController {}

//@Controller又标记了@Component元注解
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Controller {}
```

**Bean 默认是单例的，所以单例的 Controller 注入的 Service 也是一次性创建的，即使 Service 本身标识了 prototype 的范围也没用。**