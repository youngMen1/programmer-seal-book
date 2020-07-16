# 1.Dubbo扩展机制SPI
## 1.1.目标：
让读者知道JDK的SPI思想，dubbo的SPI思想，dubbo扩展机制SPI的原理，能够读懂实现扩展机制的源码。

## 1.2.基本介绍
第一篇源码分析的文章就先来讲讲dubbo扩展机制spi的原理，浏览过dubbo官方文档的朋友肯定知道，dubbo有大量的spi扩展实现，包括协议扩展、调用拦截扩展、路由扩展等26个扩展，并且spi机制运用到了各个模块设计中。所以我打算先讲解dubbo的扩展机制spi。

## 1.3.JDK的SPI思想
SPI的全名为Service Provider Interface，面向对象的设计里面，模块之间推荐基于接口编程，而不是对实现类进行硬编码，这样做也是为了模块设计的可拔插原则。为了在模块装配的时候不在程序里指明是哪个实现，就需要一种服务发现的机制，jdk的spi就是为某个接口寻找服务实现。jdk提供了服务实现查找的工具类：java.util.ServiceLoader，它会去加载META-INF/service/目录下的配置文件。具体的内部实现逻辑为这里先不展开，主要还是讲解dubbo关于spi的实现原理。

## Dubbo的SPI扩展机制原理

dubbo自己实现了一套SPI机制，改进了JDK标准的SPI机制：

JDK标准的SPI只能通过遍历来查找扩展点和实例化，有可能导致一次性加载所有的扩展点，如果不是所有的扩展点都被用到，就会导致资源的浪费。dubbo每个扩展点都有多种实现，例如com.alibaba.dubbo.rpc.Protocol接口有InjvmProtocol、DubboProtocol、RmiProtocol、HttpProtocol、HessianProtocol等实现，如果只是用到其中一个实现，可是加载了全部的实现，会导致资源的浪费。
把配置文件中扩展实现的格式修改，例如META-INF/dubbo/com.xxx.Protocol里的com.foo.XxxProtocol格式改为了xxx = com.foo.XxxProtocol这种以键值对的形式，这样做的目的是为了让我们更容易的定位到问题，比如由于第三方库不存在，无法初始化，导致无法加载扩展名（“A”），当用户配置使用A时，dubbo就会报无法加载扩展名的错误，而不是报哪些扩展名的实现加载失败以及错误原因，这是因为原来的配置格式没有把扩展名的id记录，导致dubbo无法抛出较为精准的异常，这会加大排查问题的难度。所以改成key-value的形式来进行配置。
dubbo的SPI机制增加了对IOC、AOP的支持，一个扩展点可以直接通过setter注入到其他扩展点。








# 3.总结

# 4.参考



