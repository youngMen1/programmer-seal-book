# 1.Apollo源码分析-Apollo的代码层次\(三\)

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

2176780131-5cf77c8197a23_articlex.png

#### NotFoundException

某个值找不到了 

2788800039-5cf77ce3b3bae_articlex.png

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

#### WebMvcConfig

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





















































