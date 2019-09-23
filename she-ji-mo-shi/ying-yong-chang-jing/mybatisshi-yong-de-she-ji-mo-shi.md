# Mybatis使用的设计模式

一、装饰模式

最明显的就是cache包下面的实现

Cahe、LoggingCache、LruCache、TransactionalCahe...等

以LoggingCache为例，UML图

![img](/static/image/20170628210545089.png)

![img](/static/image/20170628210109123.png)

```
Cache cache  = new LoggingCache(new PerpetualCache("cacheid"));
```

一层层包装就使得默认cache实现PerpetualCache具有附加的功能，比如上面的log功能。

二、建造者模式

BaseBuilder、XMLMapperBuilder

![img](/static/image/20170628214707239.png)

三、工厂方法

SqlSessionFactory

20170628213853842.png

四、适配器模式

Log、LogFactory

20170628213948664.png

五、模板方法

BaseExecutor、SimpleExecutor

20170628212637781.png

六、动态代理

Plugin 见7图

7、责任链模式

Interceptor、InterceptorChain

