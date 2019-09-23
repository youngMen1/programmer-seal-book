## [单例模式－－反射－－防止序列化破坏单例模式](https://www.cnblogs.com/ttylinux/p/6498822.html)

本文牵涉到的概念:

1.单例模式－－－－－－唯一最佳实现方式，使用枚举类实现

2.单例模式的几种实现，各自的缺点

3.反射;反射是如何破坏单例模式

4.序列化；序列化如何破坏单例模式

单例模式

单例模式，是指在任何时候，该类只能被实例化一次，在任何时候，访问该类的对象，对象都是同一的，只有一个。

**单例模式的实现方式：**

a.**使用类公有的静态成员来保存该唯一对象**

```
public class EagerSingleton {    
        // jvm保证在任何线程访问uniqueInstance静态变量之前一定先创建了此实例    
        public static EagerSingleton uniqueInstance = new EagerSingleton();    

        // 私有的默认构造子，保证外界无法直接实例化    
        private EagerSingleton() {    
        }    
}
```

**b.使用公有的静态成员工厂方法**

```
public class EagerSingleton {    
        // jvm保证在任何线程访问uniqueInstance静态变量之前一定先创建了此实例    
        private static EagerSingleton uniqueInstance = new EagerSingleton();    

        // 私有的默认构造子，保证外界无法直接实例化    
        private EagerSingleton() {    
        }    

        // 提供全局访问点获取唯一的实例    
        public static EagerSingleton getInstance() {    
                return uniqueInstance;    
        }    
}
//懒汉式
同步一个方法可能造成程序执行效率下降100倍，完全没有必要每次调用getInstance都加锁，事实上我们只想保证一次初始化成功，其余的快速返回而已,如果在getInstance频繁使用的地方就要考虑重新优化了.
public class LazySingleton {    
        private static LazySingleton uniqueInstance;    

        private LazySingleton() {    
        }    

        public static synchronized LazySingleton getInstance() {    
                if (uniqueInstance == null)    
                        uniqueInstance = new LazySingleton();    
                return uniqueInstance;    
        }    
}
```

3\)"双检锁"\(Double-Checked Lock\)尽量将"加锁"推迟,只在需要时"加锁"\(仅适用于[Java ](http://lib.csdn.net/base/java)5.0 以上版本,volatile保证原子操作\)  
happens-before:"什么什么一定在什么什么之前运行",也就是保证顺序性.  
现在的CPU有乱序执行的能力\(也就是指令会乱序或并行运行,可以不按我们写代码的顺序执行内存的存取过程\),并且多个CPU之间的缓存也不保证实时同步,只有上面的happens-before所规定的情况下才保证顺序性.

JVM能够根据CPU的特性\(CPU的多级缓存系统、多核处理器等\)适当的重新排序机器指令,使机器指令更符合CPU的执行特点，最大限度的发挥机器的性能.

如果没有volatile修饰符则可能出现一个线程t1的B操作和另一线程t2的C操作之间对instance的读写没有happens-before，可能会造成的现象是t1的B操作还没有完全构造成功，但t2的C已经看到instance为非空，这样t2就直接返回了未完全构造的instance的引用，t2想对instance进行操作就会出问题.

```
volatile 的功能:  
```

1. 避免编译器将变量缓存在寄存器里    
2. 避免编译器调整代码执行的顺序

优化器在用到这个变量时必须每次都小心地重新读取这个变量的值，而不是使用保存在寄存器里的备份。

```
public class DoubleCheckedLockingSingleton {    
        // java中使用双重检查锁定机制,由于Java编译器和JIT的优化的原因系统无法保证我们期望的执行次序。    
        // 在java5.0修改了内存模型,使用volatile声明的变量可以强制屏蔽编译器和JIT的优化工作    
        private volatile static DoubleCheckedLockingSingleton uniqueInstance;    
    
        private DoubleCheckedLockingSingleton() {    
        }    
    
        public static DoubleCheckedLockingSingleton getInstance() {    
                if (uniqueInstance == null) {    
                        synchronized (DoubleCheckedLockingSingleton.class) {    
                                if (uniqueInstance == null) {    
                                        uniqueInstance = new DoubleCheckedLockingSingleton();    
                                }    
                        }    
                }    
                return uniqueInstance;    
        }    
}    
```



