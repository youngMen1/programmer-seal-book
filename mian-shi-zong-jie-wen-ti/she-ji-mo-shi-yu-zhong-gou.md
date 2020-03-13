## 1、你能举例几个常见的设计模式 {#1、你能举例几个常见的设计模式}

单例模式、工厂模式、代理模式、模板方法模式

## 2、你在设计一个工厂的包的时候会遵循哪些原则？ {#2、你在设计一个工厂的包的时候会遵循哪些原则？}

设计模式的六大原则

1、开闭原则（Open Close Principle）

开闭原则就是说对扩展开放，对修改关闭。在程序需要进行拓展的时候，不能去修改原有的代码，实现一个热插拔的效果。所以一句话概括就是：为了使程序的扩展性好，易于维护和升级。想要达到这样的效果，我们需要使用接口和抽象类，后面的具体设计中我们会提到这点。

2、里氏代换原则（Liskov Substitution Principle）

里氏代换原则\(Liskov Substitution Principle LSP\)面向对象设计的基本原则之一。 里氏代换原则中说，任何基类可以出现的地方，子类一定可以出现。 LSP是继承复用的基石，只有当衍生类可以替换掉基类，软件单位的功能不受到影响时，基类才能真正被复用，而衍生类也能够在基类的基础上增加新的行为。里氏代换原则是对“开-闭”原则的补充。实现“开-闭”原则的关键步骤就是抽象化。而基类与子类的继承关系就是抽象化的具体实现，所以里氏代换原则是对实现抽象化的具体步骤的规范。—— From Baidu 百科

3、依赖倒转原则（Dependence Inversion Principle）

这个是开闭原则的基础，具体内容：面对接口编程，依赖于抽象而不依赖于具体。

4、接口隔离原则（Interface Segregation Principle）

这个原则的意思是：使用多个隔离的接口，比使用单个接口要好。还是一个降低类之间的耦合度的意思，从这儿我们看出，其实设计模式就是一个软件的设计思想，从大型软件架构出发，为了升级和维护方便。所以上文中多次出现：降低依赖，降低耦合。

5、迪米特法则（最少知道原则）（Demeter Principle）

为什么叫最少知道原则，就是说：一个实体应当尽量少的与其他实体之间发生相互作用，使得系统功能模块相对独立。

6、合成复用原则（Composite Reuse Principle）

原则是尽量使用合成/聚合的方式，而不是使用继承

## 3、你能列举一个使用了Visitor/Decorator模式的开源项目/库吗？ {#3、你能列举一个使用了Visitor-Decorator模式的开源项目-库吗？}

## 4、你在编码时最常用的设计模式有哪些？在什么场景下用？ {#4、你在编码时最常用的设计模式有哪些？在什么场景下用？}

单例模式：工程中只需要类的一个实例。

工厂模式：大量的对象需要创建，并且具有相同的接口时。

## 5、如何实现一个单例？ {#5、如何实现一个单例？}

```
public class Singleton {
  private Singleton() {}
  private static final Singleton instance = new Singleton();
  // 饿汉式
  public static Singleton getInstance() {
    return instance;
  }
}
public class Singleton {
  private Singleton() {}
  private static Singleton instance;
  // 懒汉式
  public static Singleton getInstance() {
    if (instance == null) {
      instance = new Singleton();
    }
    return instance;
  }
}
```

## 6、代理模式（动态代理） {#6、代理模式（动态代理）}

```
public class DynamicProxyHandler implements InvocationHandler {
    private Object tar;
    // 绑定委托对象，并返回代理类
    public Object bind(Object tar) {
        this.tar = tar;
        // 绑定该类实现的所有接口，取得代理类
        return Proxy.newProxyInstance(tar.getClass().getClassLoader(),
                tar.getClass().getInterfaces(),
                this);
    }
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        before();
        Object result = method.invoke(tar, args);
        after();
        return result;
    }
    private void after() {
    }
    private void before() {
    }
}
```

## 7、单例模式（懒汉模式，并发初始化如何解决，volatile与lock的使用） {#7、单例模式（懒汉模式，并发初始化如何解决，volatile与lock的使用）}

```
public class Singleton {
  private Singleton() {}
    private static volatile Singleton instance;
  // 懒汉式 线程安全
  public static Singleton getInstance() {
    if (instance == null) {
      synchronize (Singleton.class) {
        if (instance == null) {
          instance = new Singleton();
        }
      }
    }
    return instance;
  }
}
```

## 8、JDK源码里面都有些什么让你印象深刻的设计模式使用，举例看看？ {#8、JDK源码里面都有些什么让你印象深刻的设计模式使用，举例看看？}

[【设计模式】JDK源码中用到的设计模式](http://blog.csdn.net/baiye_xing/article/details/76427717)

迭代器模式：集合的遍历

工厂模式：java.util.Calendar\#getInstance\(\)

适配器模式：java.util.Arrays\#asList\(\)

## 9、设计模式UML类图与设计模式实现 {#9、设计模式UML类图与设计模式实现}

[java设计模式–UML类图](https://www.cnblogs.com/kanhaiba/p/5568594.html)

[Java开发中的23种设计模式详解](http://www.cnblogs.com/maowang1991/archive/2013/04/15/3023236.html)

