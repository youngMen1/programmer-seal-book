# jdk中的Unsafe类
Java和C++语言的一个重要区别就是Java中我们无法直接操作一块内存区域，不能像C++中那样可以自己申请内存和释放内存。Java中的Unsafe类为我们提供了类似C++手动管理内存的能力。
Unsafe类，全限定名是`sun.misc.Unsafe`，从名字中我们可以看出来这个类对普通程序员来说是“危险”的，一般应用开发者不会用到这个类。


Unsafe类是"final"的，不允许继承。且构造函数是private的:

```

/*
 * 除了实现sun.misc.Unsafe中列出的方法，还提供了其他更精细的低级别操作，与底层交互紧密
 *
 * 针对多线程中单变量的可视性，以及多变量的顺序依赖和值依赖，JDK 9 引入四种资源同步的语义，由弱到强分别是：
 * Plain, Opaque, Release[write]/Acquire[read], Volatile。【参见VarHandle】
 */
public final class Unsafe {
    
    // 单例对象
    private static final Unsafe theUnsafe = new Unsafe();

   // 构造函数是private的，不允许外部实例化
    private Unsafe() {
    }

....
}

```
因此我们无法在外部对Unsafe进行实例化。

## 获取Unsafe

Unsafe无法实例化，那么怎么获取Unsafe呢？答案就是通过反射来获取Unsafe：

```
public Unsafe getUnsafe() throws IllegalAccessException {
    Field unsafeField = Unsafe.class.getDeclaredFields()[0];
    unsafeField.setAccessible(true);
    Unsafe unsafe = (Unsafe) unsafeField.get(null);
    return unsafe;
}
```

## 主要功能
Unsafe的功能如下图：
![](/static/image/11963487-607a966eba2eed13.webp)

# 2.参考
jdk中的Unsafe类：https://www.jianshu.com/p/db8dce09232d