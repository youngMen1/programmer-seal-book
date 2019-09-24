# Spring AOP 实现原理与 CGLIB 应用 {#ibm-pagetitle-h1}

AOP（Aspect Orient Programming），作为面向对象编程的一种补充，广泛应用于处理一些具有横切性质的系统级服务，如事务管理、安全检查、缓存、对象池管理等。AOP 实现的关键就在于 AOP 框架自动创建的 AOP 代理，AOP 代理则可分为静态代理和动态代理两大类，其中静态代理是指使用 AOP 框架提供的命令进行编译，从而在编译阶段就可生成 AOP 代理类，因此也称为编译时增强；而动态代理则在运行时借助于 JDK 动态代理、CGLIB 等在内存中“临时”生成 AOP 动态代理类，因此也被称为运行时增强。

## AOP 的存在价值 {#major11}

在传统 OOP 编程里以对象为核心，整个软件系统由系列相互依赖的对象所组成，而这些对象将被抽象成一个一个的类，并允许使用类继承来管理类与类之间一般到特殊的关系。随着软件规模的增大，应用的逐渐升级，慢慢出现了一些 OOP 很难解决的问题。

我们可以通过分析、抽象出一系列具有一定属性与行为的对象，并通过这些对象之间的协作来形成一个完整的软件功能。由于对象可以继承，因此我们可以把具有相同功能或相同特性的属性抽象到一个层次分明的类结构体系中。随着软件规范的不断扩大，专业化分工越来越系列，以及 OOP 应用实践的不断增多，随之也暴露出了一些 OOP 无法很好解决的问题。

现在假设系统中有 3 段完全相似的代码，这些代码通常会采用“复制”、“粘贴”方式来完成，通过这种“复制”、“粘贴”方式开发出来的软件如图 1 所示。

##### 图 1.多个地方包含相同代码的软件 {#fig1}

image003.jpg

看到如图 1 所示的示意图，可能有的读者已经发现了这种做法的不足之处：如果有一天，图 1 中的深色代码段需要修改，那是不是要打开 3 个地方的代码进行修改？如果不是 3 个地方包含这段代码，而是 100 个地方，甚至是 1000 个地方包含这段代码段，那会是什么后果？

为了解决这个问题，我们通常会采用将如图 1 所示的深色代码部分定义成一个方法，然后在 3 个代码段中分别调用该方法即可。在这种方式下，软件系统的结构如图 2 所示。

##### 图 2 通过方法调用实现系统功能 {#fig2}

image005 \(1\).jpg

对于如图 2 所示的软件系统，如果需要修改深色部分的代码，只要修改一个地方即可，不管整个系统中有多少地方调用了该方法，程序无须修改这些地方，只需修改被调用的方法即可——通过这种方式，大大降低了软件后期维护的复杂度。

对于如图 2 所示的方法 1、方法 2、方法 3 依然需要显式调用深色方法，这样做能够解决大部分应用场景。但对于一些更特殊的情况：应用需要方法 1、方法 2、方法 3 彻底与深色方法分离——方法 1、方法 2、方法 3 无须直接调用深色方法，那如何解决？

因为软件系统需求变更是很频繁的事情，系统前期设计方法 1、方法 2、方法 3 时只实现了核心业务功能，过了一段时间，我们需要为方法 1、方法 2、方法 3 都增加事务控制；又过了一段时间，客户提出方法 1、方法 2、方法 3 需要进行用户合法性验证，只有合法的用户才能执行这些方法；又过了一段时间，客户又提出方法 1、方法 2、方法 3 应该增加日志记录；又过了一段时间，客户又提出……面对这样的情况，我们怎么办？通常有两种做法：

* 根据需求说明书，直接拒绝客户要求。
* 拥抱需求，满足客户的需求。

第一种做法显然不好，客户是上帝，我们应该尽量满足客户的需求。通常会采用第二种做法，那如何解决呢？是不是每次先定义一个新方法，然后修改方法 1、方法 2、方法 3，增加调用新方法？这样做的工作量也不小啊！我们希望有一种特殊的方法：我们只要定义该方法，无须在方法 1、方法 2、方法 3 中显式调用它，系统会“自动”执行该特殊方法。

上面想法听起来很神奇，甚至有一些不切实际，但其实是完全可以实现的，实现这个需求的技术就是 AOP。AOP 专门用于处理系统中分布于各个模块（不同方法）中的交叉关注点的问题，在 Java EE 应用中，常常通过 AOP 来处理一些具有横切性质的系统级服务，如事务管理、安全检查、缓存、对象池管理等，AOP 已经成为一种非常常用的解决方案。

## 使用 AspectJ 的编译时增强进行 AOP {#major12}

AspectJ 是一个基于 Java 语言的 AOP 框架，提供了强大的 AOP 功能，其他很多 AOP 框架都借鉴或采纳其中的一些思想。

AspectJ 是 Java 语言的一个 AOP 实现，其主要包括两个部分：第一个部分定义了如何表达、定义 AOP 编程中的语法规范，通过这套语言规范，我们可以方便地用 AOP 来解决 Java 语言中存在的交叉关注点问题；另一个部分是工具部分，包括编译器、调试工具等。

AspectJ 是最早、功能比较强大的 AOP 实现之一，对整套 AOP 机制都有较好的实现，很多其他语言的 AOP 实现，也借鉴或采纳了 AspectJ 中很多设计。在 Java 领域，AspectJ 中的很多语法结构基本上已成为 AOP 领域的标准。

下载、安装 AspectJ 比较简单，读者登录 AspectJ 官网（[http://www.eclipse.org/aspectj），即可下载到一个可执行的](http://www.eclipse.org/aspectj），即可下载到一个可执行的) JAR 包，使用 java -jar aspectj-1.x.x.jar 命令、多次单击“Next”按钮即可成功安装 AspectJ。

成功安装了 AspectJ 之后，将会在 E:\Java\AOP\aspectj1.6 路径下（AspectJ 的安装路径）看到如下文件结构：

* bin：该路径下存放了 aj、aj5、ajc、ajdoc、ajbrowser 等命令，其中 ajc 命令最常用，它的作用类似于 javac，用于对普通 Java 类进行编译时增强。
* docs：该路径下存放了 AspectJ 的使用说明、参考手册、API 文档等文档。
* lib：该路径下的 4 个 JAR 文件是 AspectJ 的核心类库。
* 相关授权文件。

一些文档、AspectJ 入门书籍，一谈到使用 AspectJ，就认为必须使用 Eclipse 工具，似乎离开了该工具就无法使用 AspectJ 了。

虽然 AspectJ 是 Eclipse 基金组织的开源项目，而且提供了 Eclipse 的 AJDT 插件（AspectJ Development Tools）来开发 AspectJ 应用，但 AspectJ 绝对无须依赖于 Eclipse 工具。

实际上，AspectJ 的用法非常简单，就像我们使用 JDK 编译、运行 Java 程序一样。下面通过一个简单的程序来示范 AspectJ 的用法，并分析 AspectJ 如何在编译时进行增强。

首先编写一个简单的 Java 类，这个 Java 类用于模拟一个业务组件。

```
public class Hello 
{ 
// 定义一个简单方法，模拟应用中的业务逻辑方法
public void sayHello(){System.out.println("Hello AspectJ!");}
// 主方法，程序的入口
public static void main(String[] args) 
{ 
Hello h = new Hello(); 
h.sayHello(); 
} 
}
```

上面 Hello 类模拟了一个业务逻辑组件，编译、运行该 Java 程序，这个结果是没有任何悬念的，程序将在控制台打印“Hello AspectJ”字符串。

假设现在客户需要在执行 sayHello\(\) 方法之前启动事务，当该方法执行结束时关闭事务，在传统编程模式下，我们必须手动修改 sayHello\(\) 方法——如果改为使用 AspectJ，则可以无须修改上面的 sayHello\(\) 方法。

下面我们定义一个特殊的 Java 类。

```
public aspect TxAspect 
{ 
// 指定执行 Hello.sayHello() 方法时执行下面代码块
void around():call(void Hello.sayHello()){System.out.println("开始事务 ...");proceed();System.out.println("事务结束 ...");}
}
```

可能读者已经发现了，上面类文件中不是使用 class、interface、enum 在定义 Java 类，而是使用了 aspect ——难道 Java 语言又新增了关键字？没有！上面的 TxAspect 根本不是一个 Java 类，所以 aspect 也不是 Java 支持的关键字，它只是 AspectJ 才能识别的关键字。

上面粗体字代码也不是方法，它只是指定当程序执行 Hello 对象的 sayHello\(\) 方法时，系统将改为执行粗体字代码的花括号代码块，其中 proceed\(\) 代表回调原来的 sayHello\(\) 方法。

正如前面提到的，Java 无法识别 TxAspect.java 文件的内容，所以我们要使用 ajc.exe 命令来编译上面的 Java 程序。为了能在命令行使用 ajc.exe 命令，需要把 AspectJ 安装目录下的 bin 路径（比如 E:\Java\AOP\aspectj1.6\bin 目录）添加到系统的 PATH 环境变量中。接下来执行如下命令进行编译：

ajc -d . Hello.java TxAspect.java

我们可以把 ajc.exe 理解成 javac.exe 命令，都用于编译 Java 程序，区别是 ajc.exe 命令可识别 AspectJ 的语法；从这个意义上看，我们可以将 ajc.exe 当成一个增强版的 javac.exe 命令。

运行该 Hello 类依然无须任何改变，因为 Hello 类位于 lee 包下。程序使用如下命令运行 Hello 类：

java lee.Hello

运行该程序，将看到一个令人惊喜的结果：

开始事务 ...

Hello AspectJ!

事务结束 ...

从上面运行结果来看，我们完全可以不对 Hello.java 类进行任何修改，同时又可以满足客户的需求：上面程序只是在控制台打印“开始事务 ...”、“结束事务 ...”来模拟了事务操作，实际上我们可用实际的事务操作代码来代替这两行简单的语句，这就可以满足客户需求了。

如果客户再次提出新需求，需要在 sayHello\(\) 方法后增加记录日志的功能，那也很简单，我们再定义一个 LogAspect，程序如下：

```
public aspect LogAspect 
{ 
// 定义一个 PointCut，其名为 logPointcut 
// 该 PointCut 对应于指定 Hello 对象的 sayHello 方法
    pointcut logPointcut() 
:execution(void Hello.sayHello()); 
// 在 logPointcut 之后执行下面代码块
    after():logPointcut() 
    { 
System.out.println("记录日志 ..."); 
    } 
}
```

上面程序的粗体字代码定义了一个 Pointcut：logPointcut - 等同于执行 Hello 对象的 sayHello\(\) 方法，并指定在 logPointcut 之后执行简单的代码块，也就是说，在 sayHello\(\) 方法之后执行指定代码块。使用如下命令来编译上面的 Java 程序：

ajc -d . \*.java

再次运行 Hello 类，将看到如下运行结果：

开始事务 ...

Hello AspectJ!

记录日志 ...

事务结束 ...

从上面运行结果来看，通过使用 AspectJ 提供的 AOP 支持，我们可以为 sayHello\(\) 方法不断增加新功能。

为什么在对 Hello 类没有任何修改的前提下，而 Hello 类能不断地、动态增加新功能呢？这看上去并不符合 Java 基本语法规则啊。实际上我们可以使用 Java 的反编译工具来反编译前面程序生成的 Hello.class 文件，发现 Hello.class 文件的代码如下：

```
package lee; 

import java.io.PrintStream; 
import org.aspectj.runtime.internal.AroundClosure; 

public class Hello 
{ 
 public void sayHello() 
 { 
   try 
   { 
     System.out.println("Hello AspectJ!"); } catch (Throwable localThrowable) { 
     LogAspect.aspectOf().ajc$after$lee_LogAspect$1$9fd5dd97(); throw localThrowable; } 
     LogAspect.aspectOf().ajc$after$lee_LogAspect$1$9fd5dd97(); 
 } 

 ... 

 private static final void sayHello_aroundBody1$advice(Hello target, 
            TxAspect ajc$aspectInstance, AroundClosure ajc$aroundClosure) 
 { 
   System.out.println("开始事务 ..."); 
   AroundClosure localAroundClosure = ajc$aroundClosure; sayHello_aroundBody0(target); 
   System.out.println("事务结束 ..."); 
 } 
}
```

不难发现这个 Hello.class 文件不是由原来的 Hello.java 文件编译得到的，该 Hello.class 里新增了很多内容——这表明 AspectJ 在编译时“自动”编译得到了一个新类，这个新类增强了原有的 Hello.java 类的功能，因此 AspectJ 通常被称为编译时增强的 AOP 框架。

提示：与 AspectJ 相对的还有另外一种 AOP 框架，它们不需要在编译时对目标类进行增强，而是运行时生成目标类的代理类，该代理类要么与目标类实现相同的接口，要么是目标类的子类——总之，代理类的实例可作为目标类的实例来使用。一般来说，编译时增强的 AOP 框架在性能上更有优势——因为运行时动态增强的 AOP 框架需要每次运行时都进行动态增强。

实际上，AspectJ 允许同时为多个方法添加新功能，只要我们定义 Pointcut 时指定匹配更多的方法即可。如下片段：

```
pointcut xxxPointcut() 
    :execution(void H*.say*());
```

上面程序中的 xxxPointcut 将可以匹配所有以 H 开头的类中、所有以 say 开头的方法，但该方法返回的必须是 void；如果不想匹配任意的返回值类型，则可将代码改为如下形式：

pointcut xxxPointcut\(\)

:execution\(\* H\*.say\*\(\)\);

关于如何定义 AspectJ 中的 Aspect、Pointcut 等，读者可以参考 AspectJ 安装路径下的 doc 目录里的 quick5.pdf 文件。

## 使用 Spring AOP {#major13}

与 AspectJ 相同的是，Spring AOP 同样需要对目标类进行增强，也就是生成新的 AOP 代理类；与 AspectJ 不同的是，Spring AOP 无需使用任何特殊命令对 Java 源代码进行编译，它采用运行时动态地、在内存中临时生成“代理类”的方式来生成 AOP 代理。

Spring 允许使用 AspectJ Annotation 用于定义方面（Aspect）、切入点（Pointcut）和增强处理（Advice），Spring 框架则可识别并根据这些 Annotation 来生成 AOP 代理。Spring 只是使用了和 AspectJ 5 一样的注解，但并没有使用 AspectJ 的编译器或者织入器（_Weaver_），底层依然使用的是 Spring AOP，依然是在运行时动态生成 AOP 代理，并不依赖于 AspectJ 的编译器或者织入器。

简单地说，Spring 依然采用运行时生成动态代理的方式来增强目标对象，所以它不需要增加额外的编译，也不需要 AspectJ 的织入器支持；而 AspectJ 在采用编译时增强，所以 AspectJ 需要使用自己的编译器来编译 Java 文件，还需要织入器。

为了启用 Spring 对 @AspectJ 方面配置的支持，并保证 Spring 容器中的目标 Bean 被一个或多个方面自动增强，必须在 Spring 配置文件中配置如下片段：

```
<?xml version="1.0" encoding="GBK"?> 
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:aop="http://www.springframework.org/schema/aop"
xsi:schemaLocation="http://www.springframework.org/schema/beans 
http://www.springframework.org/schema/beans/spring-beans-3.0.xsd 
http://www.springframework.org/schema/aop 
http://www.springframework.org/schema/aop/spring-aop-3.0.xsd"> 
<!-- 启动 @AspectJ 支持 -->
<aop:aspectj-autoproxy/> 
</beans>
```

当然，如果我们希望完全启动 Spring 的“零配置”功能，则还需要启用 Spring 的“零配置”支持，让 Spring 自动搜索指定路径下 Bean 类。

所谓自动增强，指的是 Spring 会判断一个或多个方面是否需要对指定 Bean 进行增强，并据此自动生成相应的代理，从而使得增强处理在合适的时候被调用。

如果不打算使用 Spring 的 XML Schema 配置方式，则应该在 Spring 配置文件中增加如下片段来启用 @AspectJ 支持。

```
<!-- 启动 @AspectJ 支持 -->
<bean class="org.springframework.aop.aspectj.annotation. 
    AnnotationAwareAspectJAutoProxyCreator"/>
```

上面配置文件中的 AnnotationAwareAspectJAutoProxyCreator 是一个 Bean 后处理器（BeanPostProcessor），该 Bean 后处理器将会为容器中 Bean 生成 AOP 代理，

当启动了 @AspectJ 支持后，只要我们在 Spring 容器中配置一个带 @Aspect 注释的 Bean，Spring 将会自动识别该 Bean，并将该 Bean 作为方面 Bean 处理。

在 Spring 容器中配置方面 Bean（即带 @Aspect 注释的 Bean），与配置普通 Bean 没有任何区别，一样使用 &lt;bean.../&gt; 元素进行配置，一样支持使用依赖注入来配置属性值；如果我们启动了 Spring 的“零配置”特性，一样可以让 Spring 自动搜索，并装载指定路径下的方面 Bean。

使用 @Aspect 标注一个 Java 类，该 Java 类将会作为方面 Bean，如下面代码片段所示：

```
// 使用 @Aspect 定义一个方面类
@Aspect 
public class LogAspect 
{ 
// 定义该类的其他内容
... 
}
```

方面类（用 @Aspect 修饰的类）和其他类一样可以有方法、属性定义，还可能包括切入点、增强处理定义。

当我们使用 @Aspect 来修饰一个 Java 类之后，Spring 将不会把该 Bean 当成组件 Bean 处理，因此负责自动增强的后处理 Bean 将会略过该 Bean，不会对该 Bean 进行任何增强处理。

开发时无须担心使用 @Aspect 定义的方面类被增强处理，当 Spring 容器检测到某个 Bean 类使用了 @Aspect 标注之后，Spring 容器不会对该 Bean 类进行增强。

下面将会考虑采用 Spring AOP 来改写前面介绍的例子：

下面例子使用一个简单的 Chinese 类来模拟业务逻辑组件：

```
@Component 
public class Chinese 
{ 
// 实现 Person 接口的 sayHello() 方法
    public String sayHello(String name) 
    { 
   System.out.println("-- 正在执行 sayHello 方法 --"); 
// 返回简单的字符串
        return name + " Hello , Spring AOP"; 
    } 
// 定义一个 eat() 方法
    public void eat(String food) 
    { 
   System.out.println("我正在吃 :"+ food); 
    } 
}
```

提供了上面 Chinese 类之后，接下来假设同样需要为上面 Chinese 类的每个方法增加事务控制、日志记录，此时可以考虑使用 Around、AfterReturning 两种增强处理。

先看 AfterReturning 增强处理代码。

```
// 定义一个方面
@Aspect 
public class AfterReturningAdviceTest 
{ 
// 匹配 org.crazyit.app.service.impl 包下所有类的、
// 所有方法的执行作为切入点
@AfterReturning(returning="rvt",
pointcut="execution(* org.crazyit.app.service.impl.*.*(..))")
public void log(Object rvt) 
{ 
System.out.println("获取目标方法返回值 :" + rvt); 
System.out.println("模拟记录日志功能 ..."); 
} 
}
```

上面 Aspect 类使用了 @Aspect 修饰，这样 Spring 会将它当成一个方面 Bean 进行处理。其中程序中粗体字代码指定将会在调用 org.crazyit.app.service.impl 包下的所有类的所有方法之后织入 log\(Object rvt\) 方法。

再看 Around 增强处理代码：

```
// 定义一个方面
@Aspect 
public class AroundAdviceTest 
{ 
// 匹配 org.crazyit.app.service.impl 包下所有类的、
// 所有方法的执行作为切入点
@Around("execution(* org.crazyit.app.service.impl.*.*(..))")
public Object processTx(ProceedingJoinPoint jp) 
throws java.lang.Throwable 
{ 
System.out.println("执行目标方法之前，模拟开始事务 ..."); 
// 执行目标方法，并保存目标方法执行后的返回值
Object rvt = jp.proceed(new String[]{"被改变的参数"}); 
System.out.println("执行目标方法之后，模拟结束事务 ..."); 
return rvt + " 新增的内容"; 
} 
}
```

与前面的 AfterReturning 增强处理类似的，此处同样使用了 @Aspect 来修饰前面 Bean，其中粗体字代码指定在调用 org.crazyit.app.service.impl 包下的所有类的所有方法的“前后（Around）” 织入 processTx\(ProceedingJoinPoint jp\) 方法

需要指出的是，虽然此处只介绍了 Spring AOP 的 AfterReturning、Around 两种增强处理，但实际上 Spring 还支持 Before、After、AfterThrowing 等增强处理，关于 Spring AOP 编程更多、更细致的编程细节，可以参考《轻量级 Java EE 企业应用实战》一书。

本示例采用了 Spring 的零配置来开启 Spring AOP，因此上面 Chinese 类使用了 @Component 修饰，而方面 Bean 则使用了 @Aspect 修饰，方面 Bean 中的 Advice 则分别使用了 @AfterReturning、@Around 修饰。接下来只要为 Spring 提供如下配置文件即可：

```
<?xml version="1.0" encoding="GBK"?> 
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:context="http://www.springframework.org/schema/context"
xmlns:aop="http://www.springframework.org/schema/aop"
xsi:schemaLocation="http://www.springframework.org/schema/beans 
http://www.springframework.org/schema/beans/spring-beans-3.0.xsd 
http://www.springframework.org/schema/context 
http://www.springframework.org/schema/context/spring-context-3.0.xsd 
http://www.springframework.org/schema/aop 
http://www.springframework.org/schema/aop/spring-aop-3.0.xsd"> 
<!-- 指定自动搜索 Bean 组件、自动搜索方面类 -->
<context:component-scan base-package="org.crazyit.app.service 
,org.crazyit.app.advice"> 
<context:include-filter type="annotation"
expression="org.aspectj.lang.annotation.Aspect"/> 
</context:component-scan> 
<!-- 启动 @AspectJ 支持 -->
<aop:aspectj-autoproxy/> 
</beans>
```

接下来按传统方式来获取 Spring 容器中 chinese Bean、并调用该 Bean 的两个方法，程序代码如下：

```
public class BeanTest 
{ 
public static void main(String[] args) 
{ 
// 创建 Spring 容器
ApplicationContext ctx = new 
ClassPathXmlApplicationContext("bean.xml"); 
Chinese p = ctx.getBean("chinese" ,Chinese.class); 
System.out.println(p.sayHello("张三")); 
p.eat("西瓜"); 
} 
}
```

从上面开发过程可以看出，对于 Spring AOP 而言，开发者提供的业务组件、方面 Bean 并没有任何特别的地方。只是方面 Bean 需要使用 @Aspect 修饰即可。程序不需要使用特别的编译器、织入器进行处理。

运行上面程序，将可以看到如下执行结果：

执行目标方法之前，模拟开始事务 ...

-- 正在执行 sayHello 方法 --

执行目标方法之后，模拟结束事务 ...

获取目标方法返回值 : 被改变的参数 Hello , Spring AOP 新增的内容

模拟记录日志功能 ...

被改变的参数 Hello , Spring AOP 新增的内容

执行目标方法之前，模拟开始事务 ...

我正在吃 : 被改变的参数

执行目标方法之后，模拟结束事务 ...

获取目标方法返回值 :null 新增的内容

模拟记录日志功能 ...

虽然程序是在调用 Chinese 对象的 sayHello、eat 两个方法，但从上面运行结果不难看出：实际执行的绝对不是 Chinese 对象的方法，而是 AOP 代理的方法。也就是说，Spring AOP 同样为 Chinese 类生成了 AOP 代理类。这一点可通过在程序中增加如下代码看出：

System.out.println\(p.getClass\(\)\);

上面代码可以输出 p 变量所引用对象的实现类，再次执行程序将可以看到上面代码产生 class org.crazyit.app.service.impl.Chinese$$EnhancerByCGLIB$$290441d2 的输出，这才是 p 变量所引用的对象的实现类，这个类也就是 Spring AOP 动态生成的 AOP 代理类。从 AOP 代理类的类名可以看出，AOP 代理类是由 CGLIB 来生成的。

如果将上面程序程序稍作修改：只要让上面业务逻辑类 Chinese 类实现一个任意接口——这种做法更符合 Spring 所倡导的“面向接口编程”的原则。假设程序为 Chinese 类提供如下 Person 接口，并让 Chinese 类实现该接口：

```
public interface Person 
{ 
String sayHello(String name); 
void eat(String food); 
}
```



