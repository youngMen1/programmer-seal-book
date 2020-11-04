## 1、描述一下Hibernate的三个状态？ {#1、描述一下Hibernate的三个状态？}

transient\(瞬时状态\)：new出来一个对象，还没被保存到数据库中

persistent\(持久化状态\)：对象已经保存到数据库中并且在hibernate session也存在该对象

detached\(离线状态\)：对象在数据库中存在，hibernate session不存在

## 2、Spring中Bean的生命周期。 {#2、Spring中Bean的生命周期。}

![img](/static/image/Spring Bean的生命周期.jpg)

1、Spring对Bean进行实例化（相当于程序中的new Xx\(\)）

2、Spring将值和Bean的引用注入进Bean对应的属性中

3、如果Bean实现了BeanNameAware接口，Spring将Bean的ID传递给setBeanName\(\)方法（实现BeanNameAware清主要是为了通过Bean的引用来获得Bean的ID，一般业务中是很少有用到Bean的ID的）

4、如果Bean实现了BeanFactoryAware接口，Spring将调用setBeanDactory\(BeanFactory bf\)方法并把BeanFactory容器实例作为参数传入。（实现BeanFactoryAware 主要目的是为了获取Spring容器，如Bean通过Spring容器发布事件等）

5、如果Bean实现了ApplicationContextAwaer接口，Spring容器将调用setApplicationContext\(ApplicationContext ctx\)方法，把y应用上下文作为参数传入.\(作用与BeanFactory类似都是为了获取Spring容器，不同的是Spring容器在调用setApplicationContext方法时会把它自己作为setApplicationContext 的参数传入，而Spring容器在调用setBeanDactory前需要程序员自己指定（注入）setBeanDactory里的参数BeanFactory \)

7、如果Bean实现了BeanPostProcess接口，Spring将调用它们的postProcessBeforeInitialization（预初始化）方法（作用是在Bean实例创建成功后对进行增强处理，如对Bean进行修改，增加某个功能）7.如果Bean实现了InitializingBean接口，Spring将调用它们的afterPropertiesSet方法，作用与在配置文件中对Bean使用init-method声明初始化的作用一样，都是在Bean的全部属性设置成功后执行的初始化方法。

8、如果Bean实现了BeanPostProcess接口，Spring将调用它们的postProcessAfterInitialization（后初始化）方法（作用与7的一样，只不过7是在Bean初始化前执行的，而这个是在Bean初始化后执行的，时机不同 \)

9、经过以上的工作后，Bean将一直驻留在应用上下文中给应用使用，直到应用上下文被销毁

10、如果Bean实现了DispostbleBean接口，Spring将调用它的destory方法，作用与在配置文件中对Bean使用destory-method属性的作用一样，都是在Bean实例销毁前执行的方法。

## 3、Spring AOP解决了什么问题？怎么实现的？ {#4、Spring-AOP解决了什么问题？怎么实现的？}

[Spring AOP 实现原理](http://blog.csdn.net/moreevan/article/details/11977115/)

## 4、Spring事务的传播属性是怎么回事？它会影响什么？ {#5、Spring事务的传播属性是怎么回事？它会影响什么？}

七个事务传播属性：  
PROPAGATION\_REQUIRED – 支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。

PROPAGATION\_SUPPORTS – 支持当前事务，如果当前没有事务，就以非事务方式执行。

PROPAGATION\_MANDATORY – 支持当前事务，如果当前没有事务，就抛出异常。

PROPAGATION\_REQUIRES\_NEW – 新建事务，如果当前存在事务，把当前事务挂起。

PROPAGATION\_NOT\_SUPPORTED – 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。

PROPAGATION\_NEVER – 以非事务方式执行，如果当前存在事务，则抛出异常。

PROPAGATION\_NESTED – 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则进行与PROPAGATION\_REQUIRED类似的操作。

4个事务隔离级别

读未提交（read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（serializable ）。下面我逐一为你解释：

* 读未提交是指，一个事务还没提交时，它做的变更就能被别的事务看到。
* 读提交是指，一个事务提交之后，它做的变更才会被其他事务看到。
* 可重复读是指，一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的。当然在可重复读隔离级别下，未提交变更对其他事务也是不可见的。
* 串行化，顾名思义是对于同一行记录，“写”会加“写锁”，“读”会加“读锁”。当出现读写锁冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行。

关键词：

1、幻读：事务1读取记录时事务2增加了记录并提交，事务1再次读取时可以看到事务2新增的记录；

2、不可重复读：事务1读取记录时，事务2更新了记录并提交，事务1再次读取时可以看到事务2修改后的记录；

3、脏读：事务1更新了记录，但没有提交，事务2读取了更新后的行，然后事务T1回滚，现在T2读取无效。

## 5、Spring中BeanFactory和FactoryBean有什么区别？ {#6、Spring中BeanFactory和FactoryBean有什么区别？}

BeanFactory，以Factory结尾，表示它是一个工厂类\(接口\)，用于管理Bean的一个工厂。在Spring中，BeanFactory是IOC容器的核心接口，它的职责包括：实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。

FactoryBean以Bean结尾，表示它是一个Bean，不同于普通Bean的是：它是实现了FactoryBean接口的Bean，根据该Bean的ID从BeanFactory中获取的实际上是FactoryBean的getObject\(\)返回的对象，而不是FactoryBean本身，如果要获取FactoryBean对象，请在id前面加一个&符号来获取。

## 6、Spring框架中IOC的原理是什么？ {#7、Spring框架中IOC的原理是什么？}

[Spring框架IOC和AOP的实现原理](https://www.cnblogs.com/cyhzzu/p/6644981.html)

## 7、spring的依赖注入有哪几种方式 {#8、spring的依赖注入有哪几种方式}

在Spring容器中为一个bean配置依赖注入有四种方式：

使用属性的setter方法注入 这是最常用的方式；

使用构造器注入；

使用Filed注入（用于注解方式）.

静态、实例工厂的方法注入

## 8、用Spring如何实现一个切面？ {#10、用Spring如何实现一个切面？}

[Spring AOP中pointcut expression表达式解析](http://blog.csdn.net/kkdelta/article/details/7441829)

```
@Aspect
@Component
public class LockAspect {
    @Pointcut("@annotation(xx.xx)")
    public void pointcut() {
    }
    @Around("pointcut()")
    public Object arount(ProceedingJoinPoint point) throws Throwable {
      ...
      try {
        return point.proceed();
      } catch (Exception e) {
        throw e;
      } finally {
        ...
      }
    }
}
```

## 9、Spring 如何实现数据库事务？ {#11、Spring-如何实现数据库事务？}

使用@Transactional注解或在配置文件里面配置

## 10.对spring框架怎么理解

Spring 的两大核心概念就是 IOC 和 aop​  
IOC​是 spring 的两大核心概念之一，IoC 给我们提供了一个 IOC 得 bean 容器，这个容器会为我们自动的创建对象，不需要我们手动的创建，IOC 有一个强大的功能 DI 依赖注入，我们可以通过 JAVA 代码或者 xml 配置可以把我们想要注入对象依赖的 bean 注入到里面，正是因为这个依赖注入，使得 IOC 有个非常强大的东西，就是解耦。  
举个例子，表现最明显的地方就是 jdbctemplate 如果我们要把它注入到容器里面就会需要依赖一个数据源，如果 jdbctenplate 和 druid 的数据源强耦合在一起，就会导致要用 jdbctemplate 得时候必须用 druid 得数据源，依赖注入就可以帮我们解决这个问题，他可以让我们在注入的时候依赖一个 datesource 接口就可以啦，不需要依赖具体的实现。还有在 springMVC 初始化，就会把所有的 bean 创建好，在程序运行就不需要创建 bean，提升了效率

aop 是面向切面编程，比如说我们平常编程的时候会遇到好多重复的代码，比如说事务和日志，我们需要在好多类里面同时把这些代码同时写进去，spring 的 aop 就为我们提供了一种方式我们可以把共有得代码抽象出来，然后切入到我们想要切入的类里面，极大的方便了代码的书写，提高了复用性，当然 aop 的实现是通过动态代理实现的，如果需要代理的对象有接口我们就使用 jdk 动态代理来完成，如果没有接口就使用 cglib 来实现。

