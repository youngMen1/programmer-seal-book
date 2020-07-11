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



















