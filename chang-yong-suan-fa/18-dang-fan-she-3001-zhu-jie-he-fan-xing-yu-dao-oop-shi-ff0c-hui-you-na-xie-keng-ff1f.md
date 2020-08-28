# 18 | 当反射、注解和泛型遇到OOP时，会有哪些坑？

你好，我是朱晔。今天，我们聊聊 Java 高级特性的话题，看看反射、注解和泛型遇到重载和继承时可能会产生的坑。

你可能说，业务项目中几乎都是增删改查，用到反射、注解和泛型这些高级特性的机会少之又少，没啥好学的。但我要说的是，只有学好、用好这些高级特性，才能开发出更简洁易读的代码，而且几乎所有的框架都使用了这三大高级特性。比如，要减少重复代码，就得用到反射和注解（详见第 21 讲）。

如果你从来没用过反射、注解和泛型，可以先通过官网有一个大概了解：

Java Reflection API ：`https://docs.oracle.com/javase/8/docs/technotes/guides/reflection/index.html`

Reflection Tutorials:`https://docs.oracle.com/javase/tutorial/reflect/index.html`

Annotations: `https://docs.oracle.com/javase/8/docs/technotes/guides/language/annotations.html`

Lesson: Annotations:`https://docs.oracle.com/javase/tutorial/java/annotations/index.html`

Generics:`https://docs.oracle.com/javase/8/docs/technotes/guides/language/generics.html`

Lesson: Generics：`https://docs.oracle.com/javase/tutorial/java/generics/index.html`

接下来，我们就通过几个案例，看看这三大特性结合 OOP 使用时会有哪些坑吧。

## 反射调用方法不是以传参决定重载

反射的功能包括，在运行时动态获取类和类成员定义，以及动态读取属性调用方法。也就是说，针对类动态调用方法，不管类中字段和方法怎么变动，我们都可以用相同的规则来读取信息和执行方法。因此，几乎所有的 ORM（对象关系映射）、对象映射、MVC 框架都使用了反射。

反射的起点是 Class 类，Class 类提供了各种方法帮我们查询它的信息。你可以通过这个文档，了解每一个方法的作用。



```
https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html
```
接下来，我们先看一个反射调用方法遇到重载的坑：有两个叫 age 的方法，入参分别是基本类型 int 和包装类型 Integer。


```

@Slf4j
public class ReflectionIssueApplication {
  private void age(int age) {
      log.info("int age = {}", age);
  }

  private void age(Integer age) {
      log.info("Integer age = {}", age);
  }
}
```
如果不通过反射调用，走哪个重载方法很清晰，比如传入 36 走 int 参数的重载方法，传入 Integer.valueOf(“36”) 走 Integer 重载：



```

ReflectionIssueApplication application = new ReflectionIssueApplication();
application.age(36);
application.age(Integer.valueOf("36"));
```

**但使用反射时的误区是，认为反射调用方法还是根据入参确定方法重载。**比如，使用 getDeclaredMethod 来获取 age 方法，然后传入 Integer.valueOf(“36”)：


```

getClass().getDeclaredMethod("age", Integer.TYPE).invoke(this, Integer.valueOf("36"));
```

输出的日志证明，走的是 int 重载方法：



```

14:23:09.801 [main] INFO org.geekbang.time.commonmistakes.advancedfeatures.demo1.ReflectionIssueApplication - int age = 36
```

输出的日志证明，走的是 int 重载方法：


```

14:23:09.801 [main] INFO org.geekbang.time.commonmistakes.advancedfeatures.demo1.ReflectionIssueApplication - int age = 36
```

其实，要通过反射进行方法调用，第一步就是通过方法签名来确定方法。具体到这个案例，getDeclaredMethod 传入的参数类型 Integer.TYPE 代表的是 int，所以实际执行方法时无论传的是包装类型还是基本类型，都会调用 int 入参的 age 方法。

把 Integer.TYPE 改为 Integer.class，执行的参数类型就是包装类型的 Integer。这时，无论传入的是 Integer.valueOf(“36”) 还是基本类型的 36：



```

getClass().getDeclaredMethod("age", Integer.class).invoke(this, Integer.valueOf("36"));
getClass().getDeclaredMethod("age", Integer.class).invoke(this, 36);
```
都会调用 Integer 为入参的 age 方法：



```

14:25:18.028 [main] INFO org.geekbang.time.commonmistakes.advancedfeatures.demo1.ReflectionIssueApplication - Integer age = 36
14:25:18.029 [main] INFO org.geekbang.time.commonmistakes.advancedfeatures.demo1.ReflectionIssueApplication - Integer age = 36
```

现在我们非常清楚了，反射调用方法，是以反射获取方法时传入的方法名称和参数类型来确定调用方法的。接下来，我们再来看一下反射、泛型擦除和继承结合在一起会碰撞出什么坑。
