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


## 泛型经过类型擦除多出桥接方法的坑
泛型是一种风格或范式，一般用于强类型程序设计语言，允许开发者使用类型参数替代明确的类型，实例化时再指明具体的类型。它是代码重用的有效手段，允许把一套代码应用到多种数据类型上，避免针对每一种数据类型实现重复的代码。

Java 编译器对泛型应用了强大的类型检测，如果代码违反了类型安全就会报错，可以在编译时暴露大多数泛型的编码错误。但总有一部分编码错误，比如泛型类型擦除的坑，在运行时才会暴露。接下来，我就和你分享一个案例吧。

有一个项目希望在类字段内容变动时记录日志，于是开发同学就想到定义一个泛型父类，并在父类中定义一个统一的日志记录方法，子类可以通过继承重用这个方法。代码上线后业务没啥问题，但总是出现日志重复记录的问题。开始时，我们怀疑是日志框架的问题，排查到最后才发现是泛型的问题，反复修改多次才解决了这个问题。

父类是这样的：有一个泛型占位符 T；有一个 AtomicInteger 计数器，用来记录 value 字段更新的次数，其中 value 字段是泛型 T 类型的，setValue 方法每次为 value 赋值时对计数器进行 +1 操作。我重写了 toString 方法，输出 value 字段的值和计数器的值：



```

class Parent<T> {
    //用于记录value更新的次数，模拟日志记录的逻辑
    AtomicInteger updateCount = new AtomicInteger();
    private T value;
    //重写toString，输出值和值更新次数
    @Override
    public String toString() {
        return String.format("value: %s updateCount: %d", value, updateCount.get());
    }
    //设置值
    public void setValue(T value) {
        this.value = value;
        updateCount.incrementAndGet();
    }
}
```
子类 Child1 的实现是这样的：继承父类，但没有提供父类泛型参数；定义了一个参数为 String 的 setValue 方法，通过 super.setValue 调用父类方法实现日志记录。我们也能明白，开发同学这么设计是希望覆盖父类的 setValue 实现：



```

class Child1 extends Parent {
    public void setValue(String value) {
        System.out.println("Child1.setValue called");
        super.setValue(value);
    }
}
```

在实现的时候，子类方法的调用是通过反射进行的。实例化 Child1 类型后，通过 getClass().getMethods 方法获得所有的方法；然后按照方法名过滤出 setValue 方法进行调用，传入字符串 test 作为参数：


```

Child1 child1 = new Child1();
Arrays.stream(child1.getClass().getMethods())
        .filter(method -> method.getName().equals("setValue"))
        .forEach(method -> {
            try {
                method.invoke(child1, "test");
            } catch (Exception e) {
                e.printStackTrace();
            }
        });
System.out.println(child1.toString());
```
运行代码后可以看到，虽然 Parent 的 value 字段正确设置了 test，但父类的 setValue 方法调用了两次，计数器也显示 2 而不是 1：


```

Child1.setValue called
Parent.setValue called
Parent.setValue called
value: test updateCount: 2
```
显然，两次 Parent 的 setValue 方法调用，是因为 getMethods 方法找到了两个名为 setValue 的方法，分别是父类和子类的 setValue 方法。

这个案例中，子类方法重写父类方法失败的原因，包括两方面：
* 一是，子类没有指定 String 泛型参数，父类的泛型方法 setValue(T value) 在泛型擦除后是 setValue(Object value)，子类中入参是 String 的 setValue 方法被当作了新方法；

* 二是，**子类的 setValue 方法没有增加 @Override 注解，因此编译器没能检测到重写失败的问题。这就说明，重写子类方法时，标记 @Override 是一个好习惯。**

但是，开发同学认为问题出在反射 API 使用不当，却没意识到重写失败。他查文档后发现，getMethods 方法能获得当前类和父类的所有 public 方法，而 getDeclaredMethods 只能获得当前类所有的 public、protected、package 和 private 方法。

于是，他就用 getDeclaredMethods 替代了 getMethods：



```

Arrays.stream(child1.getClass().getDeclaredMethods())
    .filter(method -> method.getName().equals("setValue"))
    .forEach(method -> {
        try {
            method.invoke(child1, "test");
        } catch (Exception e) {
            e.printStackTrace();
        }
    });
```
这样虽然能解决重复记录日志的问题，但没有解决子类方法重写父类方法失败的问题，得到如下输出：


```

Child1.setValue called
Parent.setValue called
value: test updateCount: 1
```

其实这治标不治本，其他人使用 Child1 时还是会发现有两个 setValue 方法，非常容易让人困惑。

幸好，架构师在修复上线前发现了这个问题，让开发同学重新实现了 Child2，继承 Parent 的时候提供了 String 作为泛型 T 类型，并使用 @Override 关键字注释了 setValue 方法，实现了真正有效的方法重写：


```

class Child2 extends Parent<String> {
    @Override
    public void setValue(String value) {
        System.out.println("Child2.setValue called");
        super.setValue(value);
    }
}
```

但很可惜，修复代码上线后，还是出现了日志重复记录：


```

Child2.setValue called
Parent.setValue called
Child2.setValue called
Parent.setValue called
value: test updateCount: 2
```
可以看到，这次是 Child2 类的 setValue 方法被调用了两次。开发同学惊讶地说，肯定是反射出 Bug 了，通过 getDeclaredMethods 查找到的方法一定是来自 Child2 类本身；而且，怎么看 Child2 类中也只有一个 setValue 方法，为什么还会重复呢？

调试一下可以发现，Child2 类其实有 2 个 setValue 方法，入参分别是 String 和 Object。

81116d6f11440f92757e4fe775df71b8.png

如果不通过反射来调用方法，我们确实很难发现这个问题。**其实，这就是泛型类型擦除导致的问题。**我们来分析一下。

我们知道，Java 的泛型类型在编译后擦除为 Object。虽然子类指定了父类泛型 T 类型是 String，但编译后 T 会被擦除成为 Object，所以父类 setValue 方法的入参是 Object，value 也是 Object。如果子类 Child2 的 setValue 方法要覆盖父类的 setValue 方法，那入参也必须是 Object。所以，编译器会为我们生成一个所谓的 bridge 桥接方法，你可以使用 javap 命令来反编译编译后的 Child2 类的 class 字节码：



```

javap -c /Users/zhuye/Documents/common-mistakes/target/classes/org/geekbang/time/commonmistakes/advancedfeatures/demo3/Child2.class
Compiled from "GenericAndInheritanceApplication.java"
class org.geekbang.time.commonmistakes.advancedfeatures.demo3.Child2 extends org.geekbang.time.commonmistakes.advancedfeatures.demo3.Parent<java.lang.String> {
  org.geekbang.time.commonmistakes.advancedfeatures.demo3.Child2();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method org/geekbang/time/commonmistakes/advancedfeatures/demo3/Parent."<init>":()V
       4: return


  public void setValue(java.lang.String);
    Code:
       0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
       3: ldc           #3                  // String Child2.setValue called
       5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
       8: aload_0
       9: aload_1
      10: invokespecial #5                  // Method org/geekbang/time/commonmistakes/advancedfeatures/demo3/Parent.setValue:(Ljava/lang/Object;)V
      13: return


  public void setValue(java.lang.Object);
    Code:
       0: aload_0
       1: aload_1
       2: checkcast     #6                  // class java/lang/String
       5: invokevirtual #7                  // Method setValue:(Ljava/lang/String;)V
       8: return
}
```


