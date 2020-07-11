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










