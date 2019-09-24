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

4\)Lazy initialization holder class 满足所有 Double-Checked Locking 满足的条件，并且没有显示的同步操作

```
public class LazyInitHolderSingleton {    
        private LazyInitHolderSingleton() {    
        }    

        private static class SingletonHolder {    
                private static final LazyInitHolderSingleton INSTANCE = new LazyInitHolderSingleton();    
        }    

        public static LazyInitHolderSingleton getInstance() {    
                return SingletonHolder.INSTANCE;    
        }    
}
```

根据jvm规范，当某对象第一次调用LazyInitHolderSingleton.getInstance\(\)时，LazyInitHolderSingleton类被首次主动使用，jvm对其进行初始化（此时并不会调用LazyInitHolderSingleton\(\)构造方法;进行LazyInitHolderSingleton的类加载，初始化静态变量），然后LazyInitHolderSingleton调用getInstance\(\)方法，该方法中，又首次主动使用了SingletonHolder类，所以要对SingletonHolder类进行初始化（类的静态变量首先加载，进行初始化），初始化中，INSTANCE常量被赋值时才调用了 LazyInitHolderSingleton的构造方法LazyInitHolderSingleton\(\)，完成了实例化并返回该实例。

当再有对象（也许是在别的线程中）再次调用LazyInitHolderSingleton.getInstance\(\)时，因为已经初始化过了，不会再进行初始化步骤，所以直接返回INSTANCE常量即同一个LazyInitHolderSingleton实例。

**C　使用枚举类的方式来实现单例**

**推荐做法**

```
public enum SingletonClass {
INSTANCE;

private String name;
public void test() {
System.out.println("The Test!");
}

public void setName(String name){

this.name= name;
}

public String getName(){

return name;
}
}

public class TestMain {

public static void main(String[] args) {

SingletonClass one = SingletonClass.INSTANCE;
SingletonClass two = SingletonClass.INSTANCE;


one.test();
one.setName("I am a SingletonClass Instance");
System.out.println(one.getName());

if (one == two) {

System.out.println("There are same");
}
}
}
```

**反射**

反射是如何破坏单例模式的，单例模式的目标是，任何时候该类都只有唯一的一个对象

比如，实现一个单例:

```
public class Elvis  
{  
    private static boolean flag = false;  

    private Elvis(){  
    }  

    private  static class SingletonHolder{  
        private static final Elvis INSTANCE = new Elvis();  
    }  

    public static Elvis getInstance()  
    {  
        return SingletonHolder.INSTANCE;  
    }  

    public void doSomethingElse()  
    {  

    }  
}
```

使用反射的方式来实例化该类

```
package com.effective.singleton;  

import java.lang.reflect.Constructor;  
import java.lang.reflect.InvocationTargetException;  

public class ElvisReflectAttack  
{  

    public static void main(String[] args) throws InstantiationException, IllegalAccessException, IllegalArgumentException, InvocationTargetException, NoSuchMethodException, SecurityException  
    {  
        Class<?> classType = Elvis.class;  

        Constructor<?> c = classType.getDeclaredConstructor(null);  
        c.setAccessible(true);  
        Elvis e1 = (Elvis)c.newInstance();  
        Elvis e2 = Elvis.getInstance();  
        System.out.println(e1==e2);  
    }  

}
```

**输出结果为false，说明e1和e2不是同一个对象，到这里，单例模式不起作用**

。如果e1和e2都是指向同一个对象的，那么它们的引用值相等。

**序列化**

使用序列化的方式，单例模式是如何失效的

