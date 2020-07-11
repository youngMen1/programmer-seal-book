\`\`\# 1.Apollo源码分析-Apollo的代码层次\(三\)

## 1.1.基本介绍

不同与其它中间件框架，Apollo中有大量的业务代码，它向我们展示了大神是如何写业务代码的：maven依赖的层次结构，如何进行基础包配置，以及工具类编写，可以称之为springboot之最佳实践。

## 1.2.apollo项目依赖

apollo中有7个子项目  
最重要的有四个  
apollo-portal:后台管理服务  
apollo-admin:后台配置管理服务，用户发布了的配置项会经过portal-&gt;admin更新到数据库  
apollo-configservice: 配置管理服务，客户端通过该服务拉取配置项  
apollo-client:客户端，集成该客户端拉取配置项  
此外还有apollo-biz，apollo-common,apollo-core提供基础服务

其依赖关系如下:  
![](/static/image/2051242107-5cf737fd43e95_articlex.jpg)

## 1.3.apollo-common分析

![](/static/image/304542458-5cf739653eaa7_articlex.jpg)

### 1.3.1.utils 基础包

utils中集成了了一些通用方法，比如判断非空，对象拷贝，字符串拼接等

#### BeanUtils 拷贝对象

实现不同类对象中属性的拷贝，服务之间传递的都是dto对象，而在使用时必须转换为用法:

```
//在网络中传输的为DTO对象，而程序中处理的是实体类对象
@RequestMapping(path = "/apps", method = RequestMethod.POST)
public AppDTO create(@RequestBody AppDTO dto) {
 ...
//DTO拷贝成实体类
App entity = BeanUtils.transfrom(App.class, dto);
...
//实体类再拷贝成DTO   
dto = BeanUtils.transfrom(AppDTO.class, entity);
```

源码:

```
// 封装{@link org.springframework.beans.BeanUtils#copyProperties}，惯用与直接将转换结果返回
  public static <T> T transfrom(Class<T> clazz, Object src) {
    if (src == null) {
      return null;
    }
    T instance = null;
    try {
      instance = clazz.newInstance();
    } catch (Exception e) {
      throw new BeanUtilsException(e);
    }
    org.springframework.beans.BeanUtils.copyProperties(src, instance, getNullPropertyNames(src));
    return instance;
  }
```

#### ExceptionUtils 将exception转为String

```
//将exception转为String
 } catch (IllegalAccessException ex) {
    if (logger.isErrorEnabled()) {
        logger.error(ExceptionUtils.exceptionToString(ex));
```

ExceptionUtils源码

```
 public static String toString(HttpStatusCodeException e) {
    Map<String, Object> errorAttributes = gson.fromJson(e.getResponseBodyAsString(), mapType);
    if (errorAttributes != null) {
      return MoreObjects.toStringHelper(HttpStatusCodeException.class).omitNullValues()
          .add("status", errorAttributes.get("status"))
          .add("message", errorAttributes.get("message"))
```

当中用到了Guava的MoreObjects的链式调用来优雅的拼接字符串，[参考Guava Object的使用](https://github.com/google/guava/wiki/CommonObjectUtilitiesExplained)

#### InputValidator

验证ClusterName和AppName是否正确

#### RequestPrecondition

做非空、正数判断等，抽象出了一个类，而不用硬编码了

```
 RequestPrecondition.checkArguments(!StringUtils.isContainEmpty(model.getReleasedBy(), model
            .getReleaseTitle()),
        "Params(releaseTitle and releasedBy) can not be empty");
```

#### UniqueKeyGenerator

key值生成器

![](/static/image/微信截图_20200711135712.png)

### 1.3.2.exception 异常包

封装常用的异常处理类，对常见的异常做了分类，比如业务异常，服务异常，not found异常等，大家做异常时不妨参考下其对异常的分类。

#### AbstractApolloHttpException

apollo异常基类，设置了httpstatus，便于返回准确的http的报错信息，其继承了RuntimeException，并加入了一个httpStatus

```
public abstract class AbstractApolloHttpException extends RuntimeException{
      protected HttpStatus httpStatus;
      ...
}
```

#### BadRequestException

业务异常类，下图可以看出其对业务异常的分类描述

![](/static/image/2176780131-5cf77c8197a23_articlex.png)

#### NotFoundException

某个值找不到了

![](/static/image/2788800039-5cf77ce3b3bae_articlex.png)

#### ServiceException

当对应的服务不可达，比如这段

```
 ServiceException e = new ServiceException(String.format("No available admin server."
                                                              + " Maybe because of meta server down or all admin server down. "
                                                              + "Meta server address: %s",
                                                              MetaDomainConsts.getDomain(env)));
```

#### BeanUtilsException

BeanUtils中进行对象转换时发生异常类

### 1.3.Controller包

封装了异常处理中心，报文转换，http序列换等工具作者

#### 1.3.1.WebMvcConfig

实现了WebMvcConfigurer和WebServerFactoryCustomizer

```
public class WebMvcConfig implements WebMvcConfigurer, WebServerFactoryCustomizer<TomcatServletWebServerFactory> {
}
```

而我们的WebMvcConfigurer是个接口，类实现这个接口来具备一定的能力，以下就列出了这些能力

![](/static/image/3894336015-5cfa6908111d8_articlex.png)

挑重点介绍下

##### HandlerMethodArgumentResolver

```
 @Override
  public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
    PageableHandlerMethodArgumentResolver pageResolver =
            new PageableHandlerMethodArgumentResolver();
    pageResolver.setFallbackPageable(PageRequest.of(0, 10));

    argumentResolvers.add(pageResolver);
  }
```

重载HandlerMethodArgumentResolver是做啥用的呢？简单来说就是用来处理spring mvc中各类参数，比如@RequestParam、@RequestHeader、@RequestBody、@PathVariable、@ModelAttribute

而是使用了addArgumentResolvers后就加入了新的参数处理能力。HandlerMethodArgumentResolver中有两个最重要的参数

```
supportsParameter：用于判定是否需要处理该参数分解，返回true为需要，并会去调用下面的方法resolveArgument。
resolveArgument：真正用于处理参数分解的方法，返回的Object就是controller方法上的形参对象。
```

比如apollo就加入的是对分页的处理: PageableHandlerMethodArgumentResolver

这里我们可以看个例子，有这样一个业务场景，用户传的报文在网络中做了加密处理，需要对用户报文做解密，相当一个公共处理逻辑，写到业务代码中不方便维护，此时就可以增加一个HandlerMethodArgumentResolver用于解密。代码参考github:xxx

##### configureContentNegotiation

```
@Override
  public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
    configurer.favorPathExtension(false);
    configurer.ignoreAcceptHeader(true).defaultContentType(MediaType.APPLICATION_JSON);
  }
```

视图解析器，这里的配置指的是不检查accept头，而且默认请求为json格式。

##### addResourceHandlers

静态资源控制器

```
@Override
  public void addResourceHandlers(ResourceHandlerRegistry registry) {
    // 10 days
    addCacheControl(registry, "img", 864000);
    addCacheControl(registry, "vendor", 864000);
    // 1 day
    addCacheControl(registry, "scripts", 86400);
    addCacheControl(registry, "styles", 86400);
    addCacheControl(registry, "views", 86400);
  }
```

静态资源的访问时间。

##### WebServerFactoryCustomizer

定制tomcat，spring boot集成了tomcat，在2.0以上版本中，通过实现WebServerFactoryCustomizer类来自定义tomcat，比如在这里设置字符集

```
@Override
  public void customize(TomcatServletWebServerFactory factory) {
    MimeMappings mappings = new MimeMappings(MimeMappings.DEFAULT);
    mappings.add("html", "text/html;charset=utf-8");
    factory.setMimeMappings(mappings );

  }
```

#### 1.3.2.GlobalDefaultExceptionHandler

统一异常处理类，用于抓取controller层的所有异常，从此再也不用写超级多的try...catch了。只要加了@ControllerAdvice就能抓取所有异常了。

```
@ControllerAdvice
public class GlobalDefaultExceptionHandler {
.......
}
```

而后使用@ExcepionHandler来抓取异常，比如这样

```
  // 处理系统内置的Exception
  @ExceptionHandler(Throwable.class)
  public ResponseEntity<Map<String, Object>> exception(HttpServletRequest request, Throwable ex) {
    return handleError(request, INTERNAL_SERVER_ERROR, ex);
  }
```

在apollo中定义了这几个异常:  
内置异常: `Throwable,HttpRequestMethodNotSupportedException,HttpStatusCodeException,AccessDeniedException`  
以及apollo自定义的异常AbstractApolloHttpException  
将异常进行分类能方便直观的展示所遇到的异常

#### 1.3.3.HttpMessageConverters

根据用户请求头来格式化不同对象。请求传给服务器的都是一个字符串流，而服务器根据用户请求头判断不同媒体类型，然后在已注册的转换器中查找对应的转换器，比如在content-type中发现json后，就能转换成json对象了



```
@Configuration
public class HttpMessageConverterConfiguration {
  @Bean
  public HttpMessageConverters messageConverters() {
    GsonHttpMessageConverter gsonHttpMessageConverter = new GsonHttpMessageConverter();
    gsonHttpMessageConverter.setGson(
            new GsonBuilder().setDateFormat("yyyy-MM-dd'T'HH:mm:ss.SSSZ").create());
    final List<HttpMessageConverter<?>> converters = Lists.newArrayList(
            new ByteArrayHttpMessageConverter(), new StringHttpMessageConverter(),
            new AllEncompassingFormHttpMessageConverter(), gsonHttpMessageConverter);
    return new HttpMessageConverters() {
      @Override
      public List<HttpMessageConverter<?>> getConverters() {
        return converters;
      }
    };
  }
}
```
apollo中自定了GsonHttpMessageConverter,重写了默认的json转换器，这种转换当然更快乐，Gson是google的一个json转换器，当然，传说ali 的fastjson会更快，但是貌似fastjson问题会很多。json处理中对于日期格式的处理也是一个大问题，所以这里也定义了日期格式转换器。


#### 1.3.4.CharacterEncodingFilterConfiguration 过滤器

```
@Configuration
public class CharacterEncodingFilterConfiguration {

  @Bean
  public FilterRegistrationBean encodingFilter() {
    FilterRegistrationBean bean = new FilterRegistrationBean();
    bean.setFilter(new CharacterEncodingFilter());
    bean.addInitParameter("encoding", "UTF-8");
    bean.setName("encodingFilter");
    bean.addUrlPatterns("/*");
    bean.setDispatcherTypes(DispatcherType.REQUEST, DispatcherType.FORWARD);
    return bean;
  }
}
```
加入了一个CharacterEncodingFilter将所有的字符集全部转换成UTF-8.

### 1.4.aop包

里面只定义了一个类，用于给所有的数据库操作都加上cat链路跟踪，简单看下它的用法



```
@Aspect  //定义一个切面
@Component
public class RepositoryAspect {
/**
** 所有Repository下的类都必须都添加切面
*/
  @Pointcut("execution(public * org.springframework.data.repository.Repository+.*(..))")
  public void anyRepositoryMethod() {
  }
/**
** 切面的具体方法
*/
  @Around("anyRepositoryMethod()")
  public Object invokeWithCatTransaction(ProceedingJoinPoint joinPoint) throws Throwable {
    ...
}

```
### 1.5.condition 条件注解

cloud条件注解

```
@ConditionalOnBean：当SpringIoc容器内存在指定Bean的条件
@ConditionalOnClass：当SpringIoc容器内存在指定Class的条件
@ConditionalOnExpression：基于SpEL表达式作为判断条件
@ConditionalOnJava：基于JVM版本作为判断条件
@ConditionalOnJndi：在JNDI存在时查找指定的位置
@ConditionalOnMissingBean：当SpringIoc容器内不存在指定Bean的条件
@ConditionalOnMissingClass：当SpringIoc容器内不存在指定Class的条件
@ConditionalOnNotWebApplication：当前项目不是Web项目的条件
@ConditionalOnProperty：指定的属性是否有指定的值
@ConditionalOnResource：类路径是否有指定的值
@ConditionalOnSingleCandidate：当指定Bean在SpringIoc容器内只有一个，或者虽然有多个但是指定首选的Bean
@ConditionalOnWebApplication：当前项目是Web项目的条件

```

ConditionalOnBean

```
@Configuration
public class Configuration1 {

    @Bean
    @ConditionalOnBean(Bean2.class)
    public Bean1 bean1() {
        return new Bean1();
    }
}

@Configuration
public class Configuration2 {

@Bean
public Bean2 bean2(){
    return new Bean2();
}

```

在spring ioc的过程中，优先解析@Component，@Service，@Controller注解的类。其次解析配置类，也就是@Configuration标注的类。最后开始解析配置类中定义的bean。

在apollo中使用自定义condition:

用注解实现spi


```
  @Configuration
  @Profile("ctrip")
  public static class CtripEmailConfiguration {

    @Bean
    public EmailService ctripEmailService() {
      return new CtripEmailService();
    }

    @Bean
    public CtripEmailRequestBuilder emailRequestBuilder() {
      return new CtripEmailRequestBuilder();
    }
  }
```

spi的定义: SPI 全称为 Service Provider Interface，是一种服务发现机制。SPI 的本质是将接口实现类的全限定名配置在文件中，并由服务加载器读取配置文件，加载实现类。这样可以在运行时，动态为接口替换实现类。正因此特性，我们可以很容易的通过 SPI 机制为我们的程序提供拓展功能。

而 **@Profile("ctrip")**是特指在系统环境变量中存在ctrip时才会生效，限定了方法的生效环境。
还有一种常见的方式是做数据库配置，比如在不同的dev，stg，prd环境中配置不同的地址，或者使用不同的数据库:

```
  @Profile("dev")
  @Profile("stg")
  @Profile("prd")
```

但这样存在一个问题是无法满足devops的一次编译，多处运行的原则，因此最好是将配置放置与外部，通过不同环境特征来获取不同的配置文件。

看下自定义的condition是如何实现的:

定义注解

```
@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(OnProfileCondition.class)   //具体的实现类
public @interface ConditionalOnMissingProfile {    //注解的名称
  /**
   * The profiles that should be inactive
   * @return
   */
  String[] value() default {};
}

```
而后再实现类中实现了对环境变量的判断



```
//实现condition接口
public class OnProfileCondition implements Condition {
  //如果match则返回true
  @Override
  public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
    //获取环境变量中所有的active值, spring.profile.active=xxx
    Set<String> activeProfiles = Sets.newHashSet(context.getEnvironment().getActiveProfiles());
    //获取profile中的所有制
    Set<String> requiredActiveProfiles = retrieveAnnotatedProfiles(metadata, ConditionalOnProfile.class.getName());
    Set<String> requiredInactiveProfiles = retrieveAnnotatedProfiles(metadata, ConditionalOnMissingProfile.class
        .getName());

    return Sets.difference(requiredActiveProfiles, activeProfiles).isEmpty()
        && Sets.intersection(requiredInactiveProfiles, activeProfiles).isEmpty();
  }

  private Set<String> retrieveAnnotatedProfiles(AnnotatedTypeMetadata metadata, String annotationType) {
    if (!metadata.isAnnotated(annotationType)) {
      return Collections.emptySet();
    }

    MultiValueMap<String, Object> attributes = metadata.getAllAnnotationAttributes(annotationType);

    if (attributes == null) {
      return Collections.emptySet();
    }

    Set<String> profiles = Sets.newHashSet();
    List<?> values = attributes.get("value");

    if (values != null) {
      for (Object value : values) {
        if (value instanceof String[]) {
          Collections.addAll(profiles, (String[]) value);
        }
        else {
          profiles.add((String) value);
        }
      }
    }

    return profiles;
```


































