# 1.Apollo源码分析-Apollo简介和架构演进(二)
## 1.1.基本介绍

Apollo是我现在看起来最"简单"的源码

不会像spring封装了一层又一层，把人绕晕，而apollo没有那么多封装，上手快，我们学习就应该从简单的开始,凭什么非要去学封的像粽子一样的spring源码，我们就是要去学简简单单，平时朴素，接地气的源码

最接近业务代码的源码。而且里面有大量的业务代码，而我们其实平时写的最多的就是业务，它向我们展示了大神是如何来写业务的，如何来配置maven依赖，如何来处理异常，如何来输入参数判断

比如它的maven依赖层次结构

![](/static/image/apollo123456.png)

它使用不同的异常处理类

![](/static/image/微信截图_20200711113149.png)

它对DTO,排序等的使用

```
 @RequestMapping(value = "/apps/{appId}/envs/{env}/clusters/{clusterName}/namespaces/{namespaceName}/items", method = RequestMethod.GET)
 public List<ItemDTO> findItems(@PathVariable String appId, @PathVariable String env,
                             @PathVariable String clusterName, @PathVariable String namespaceName,
                             @RequestParam(defaultValue = "lineNum") String orderBy) {

List<ItemDTO> items = configService.findItems(appId, Env.valueOf(env), clusterName, namespaceName);
if ("lastModifiedTime".equals(orderBy)) {
  Collections.sort(items, (o1, o2) -> {

```
平凡而接地气，让我们有了学习的榜样

当然也有比较"高端"的，比如servlet3.0中的DeferredResult用来保持长连接

```
@RequestMapping(method = RequestMethod.GET)
public DeferredResult<ResponseEntity<List<ApolloConfigNotification>>> pollNotification(

```

自定义注解


```
@Import(ApolloConfigRegistrar.class)  //用import注解来载入bean
public @interface EnableApolloConfig {

//用ImportBeanDefinitionRegistrar来注册bean
public class ApolloConfigRegistrar implements ImportBeanDefinitionRegistrar {
  @Override
  public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

```

学习知识总是从皮毛开始，学产品也是从产品功能开始，让我们先来看下apollo的应用场景

## 1.2.应用场景

### 传统开发

小A是XX团队主力开发，有一天产品说要上线一个迪士尼门票内购功能，由于迪士尼门票很火爆，产品一拍脑袋说，每个用户限购5张！

代码如下:



```
private static final int MAX_QTY_PER_USER = 5; //产品需求限购5张
if (qty > MAX_QTY_PER_USER ) {
    throw new IllegalStateException(
        String.format("每个用户最多购买%d张!", MAX_QTY_PER_USER ));
}

```

第二天中午，由于内购实在太火爆，产品急匆匆的跑过来对小A说，赶紧改成每人1张！
小A只好放弃了午饭，改代码、回归测试、上线，整整花了1个小时才搞定。。。

### Apollo开发

小B是YY团队主力开发，有一天产品说要上线一个欢乐谷门票抢购功能，由于欢乐谷门票很火爆，产品一拍脑袋说，每个用户限购5张！
小B吸取了小A的教训，二话不说把配置写在了Apollo配置中心
1358582139-5cd8dbc2c0f5f_articlex.png
第二天中午，由于内购实在太火爆，产品急匆匆的跑过来对小B说，赶紧改成每人1张！
小B不紧不慢的说：10秒内搞定

作者：方老司
链接：https://segmentfault.com/a/1190000019381816
来源：SegmentFault 思否
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。































