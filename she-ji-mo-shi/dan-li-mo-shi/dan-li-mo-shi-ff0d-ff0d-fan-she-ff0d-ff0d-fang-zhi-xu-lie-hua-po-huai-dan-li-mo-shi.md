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



