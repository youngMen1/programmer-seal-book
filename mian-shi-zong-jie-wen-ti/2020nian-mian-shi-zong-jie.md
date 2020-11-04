# 2020年面试总结

## 1.java反射机制的理解

就是在程序运行期间，能够动态知道每个类的方法和属性，并且对于每一个对象都能够调用该对象的方法和属性，这种动态的获取的机制叫做反射。

反射用到的就是java中类的Class对象，只要一个类被类加载器加载，那么Class对象就会zai 在jvm中存在，且有且只有一个。反射就是依靠这个Class对象来实现的。可以获得field，method和constructor对象，分别代表了类中的属性域，方法和构造器；有了这些就可以用反射来做很多事了

## 2.说说springboot的自动装配

简单的说 springboot 的自动装载机制就是通过配置@EnableAutoConfiguration 将配置为@Configuration 下的@Bean 方法加载到 spring 容器中，这个过程就是 spring 自动装载机制。

首先 springboot 自动装配功能是为了满足啥?是为了满足其他的插件进行扩展，因为有很多外部的 bean 我们没法管理，也不知道具体包的路径，这个时候 springboot 提供了自动装配功能，让我们外部的类能够注入到 spring 项目中。

第二，如果说这个是 springboot 的自动装配功能不如说是 spring 的自动装配功能。因为 springboot 使用了 spring3.1 出来的 ImportSelector 动态 bean 的装载实现的自动装载机制，同时使用了 META-INF/spring.factories 中的 SPI 机制实现了 spring 自动扫描到自动装载的 bean 的机制。后面再说几点，spring 发展是由 XML 文件到注解方式的一个循序渐进的过程,比如@Component 以及它的派生注解@Controller 等，

最后 spring 直接把 XML 文件变成了@Configuration 注解，这样理解就可以理解成 springboot 自动装载机制是把外部的 xml 文件的 Bean 配置导入到了自己的项目中，让 Bean 在自己的项目中运行。而起到关键作用的@EnableAutoConfiguration 只是作为了一个中介者的作用。