## 应用场景

* [《细数JDK里的设计模式》](http://blog.jobbole.com/62314/)

  * 结构型模式：

    * 适配器：用来把一个接口转化成另一个接口，如 java.util.Arrays\#asList\(\)。
    * 桥接模式：这个模式将抽象和抽象操作的实现进行了解耦，这样使得抽象和实现可以独立地变化，如JDBC；
    * 组合模式：使得客户端看来单个对象和对象的组合是同等的。换句话说，某个类型的方法同时也接受自身类型作为参数，如 Map.putAll，List.addAll、Set.addAll。
    * 装饰者模式：动态的给一个对象附加额外的功能，这也是子类的一种替代方式，如 java.util.Collections\#checkedList\|Map\|Set\|SortedSet\|SortedMap。
    * 享元模式：使用缓存来加速大量小对象的访问时间，如 valueOf\(int\)。
    * 代理模式：代理模式是用一个简单的对象来代替一个复杂的或者创建耗时的对象，如 java.lang.reflect.Proxy

  * 创建模式:

    * 抽象工厂模式：抽象工厂模式提供了一个协议来生成一系列的相关或者独立的对象，而不用指定具体对象的类型，如 java.util.Calendar\#getInstance\(\)。
    * 建造模式\(Builder\)：定义了一个新的类来构建另一个类的实例，以简化复杂对象的创建，如：java.lang.StringBuilder\#append\(\)。
    * 工厂方法：就是
      **一个返**
      \* 回具体对象的方法，而不是多个，如 java.lang.Object\#toString\(\)、java.lang.Class\#newInstance\(\)。
    * 原型模式：使得类的实例能够生成自身的拷贝、如：java.lang.Object\#clone\(\)。
    * 单例模式：全局只有一个实例，如 java.lang.Runtime\#getRuntime\(\)。

  * 行为模式：

    * 责任链模式：通过把请求从一个对象传递到链条中下一个对象的方式，直到请求被处理完毕，以实现对象间的解耦。如 javax.servlet.Filter\#doFilter\(\)。
    * 命令模式：将操作封装到对象内，以便存储，传递和返回，如：java.lang.Runnable。
    * 解释器模式：定义了一个语言的语法，然后解析相应语法的语句，如，java.text.Format，java.text.Normalizer。
    * 迭代器模式：提供一个一致的方法来顺序访问集合中的对象，如 java.util.Iterator。
    * 中介者模式：通过使用一个中间对象来进行消息分发以及减少类之间的直接依赖，java.lang.reflect.Method\#invoke\(\)。
    * 空对象模式：如 java.util.Collections\#emptyList\(\)。
    * 观察者模式：它使得一个对象可以灵活的将消息发送给感兴趣的对象，如 java.util.EventListener。
    * 模板方法模式：让子类可以重写方法的一部分，而不是整个重写，如 java.util.Collections\#sort\(\)。

* [《Spring-涉及到的设计模式汇总》](https://www.cnblogs.com/hwaggLee/p/4510687.html)

* [《Mybatis使用的设计模式》](https://blog.csdn.net/u012387062/article/details/54719114)



