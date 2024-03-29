## 1. 三大性质简介

在并发编程中分析线程安全的问题时往往需要切入点，那就是\*\*两大核心\*\*：JMM抽象内存模型以及happens-before规则（在\[这篇文章，就想着将并发编程中这两大神器在 \*\*原子性，有序性和可见性\*\*上做一个比较，当然这也是面试中的高频考点，值得注意。

## 2. 原子性

原子性是指\*\***一个操作是不可中断的，要么全部执行成功要么全部执行失败，有着“同生共死”的感觉**\*\*。及时在多个线程一起执行的时候，一个操作一旦开始，就不会被其他线程所干扰。我们先来看看哪些是原子操作，哪些不是原子操作，有一个直观的印象：

```
> int a = 10;  //1
> 
> a++;  //2
> 
> int b=a; //3
> 
> a = a+1; //4
```

上面这四个语句中只\*\***有第1个语句是原子操作**\*\*，将10赋值给线程工作内存的变量a,而语句2（a++），实际上包含了三个操作：1. 读取变量a的值；2：对a进行加一的操作；3.将计算后的值再赋值给变量a，而这三个操作无法构成原子操作。对语句3,4的分析同理可得这两条语句不具备原子性。当然，\[java内存模型\]\([https://juejin.im/post/5ae6d309518825673123fd0e\)中定义了8中操作都是原子的，不可再分的。](https://juejin.im/post/5ae6d309518825673123fd0e%29中定义了8中操作都是原子的，不可再分的。)

1. lock\(锁定\)：作用于主内存中的变量，它把一个变量标识为一个线程独占的状态；

2. unlock\(解锁\):作用于主内存中的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定

3. read（读取）：作用于主内存的变量，它把一个变量的值从主内存传输到线程的工作内存中，以便后面的load动作使用；

4. load（载入）：作用于工作内存中的变量，它把read操作从主内存中得到的变量值放入工作内存中的变量副本

5. use（使用）：作用于工作内存中的变量，它把工作内存中一个变量的值传递给执行引擎，每当虚拟机遇到一个需要使用到变量的值的字节码指令时将会执行这个操作；

6. assign（赋值）：作用于工作内存中的变量，它把一个从执行引擎接收到的值赋给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作；

7. store（存储）：作用于工作内存的变量，它把工作内存中一个变量的值传送给主内存中以便随后的write操作使用；

8. write（操作）：作用于主内存的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中。

上面的这些指令操作是相当底层的，可以作为扩展知识面掌握下。那么如何理解这些指令了?比如，把一个变量从主内存中复制到工作内存中就需要执行read,load操作，将工作内存同步到主内存中就需要执行store,write操作。注意的是：\*\***java内存模型只是要求上述两个操作是顺序执行的并不是连续执行的**\*\*。也就是说read和load之间可以插入其他指令，store和writer可以插入其他指令。比如对主内存中的a,b进行访问就可以出现这样的操作顺序：\*\***read a,read b, load b,load a**\*\*。由原子性变量操作read,load,use,assign,store,write，可以\*\*大致认为基本数据类型的访问读写具备原子性\*\*（例外就是long和double的非原子性协定）

&gt; **synchronized**

上面一共有八条原子操作，其中六条可以满足基本数据类型的访问读写具备原子性，还剩下lock和unlock两条原子操作。如果我们需要更大范围的原子性操作就可以使用lock和unlock原子操作。尽管jvm没有把lock和unlock开放给我们使用，但jvm以更高层次的指令monitorenter和monitorexit指令开放给我们使用，反应到java代码中就是---synchronized关键字，也就是说\*\***synchronized满足原子性**\*\*。

**&gt; volatile**

我们先来看这样一个例子：

```
public class VolatileExample {
        private static volatile int counter = 0;

        public static void main(String[] args) {
            for (int i = 0; i < 10; i++) {
                Thread thread = new Thread(new Runnable() {
                    @Override
                    public void run() {
                        for (int i = 0; i < 10000; i++)
                            counter++;
                    }
                });
                thread.start();
            }
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(counter);
        }
    }
```

开启10个线程，每个线程都自加10000次，如果不出现线程安全的问题最终的结果应该就是：10\*10000 = 100000;可是运行多次都是小于

100000的结果，问题在于 \*\***volatile并不能保证原子性**\*\*，在前面说过counter++这并不是一个原子操作，包含了三个步骤：1.读取变量

counter的值；2.对counter加一；3.将新值赋值给变量counter。如果线程A读取counter到工作内存后，其他线程对这个值已经做了自增操作

后，那么线程A的这个值自然而然就是一个过期的值，因此，总结果必然会是小于100000的。

如果让volatile保证原子性，必须符合以下两条规则：

1. \*\***运算结果并不依赖于变量的当前值，或者能够确保只有一个线程修改变量的值；**\*\*

2. \*\***变量不需要与其他的状态变量共同参与不变约束**\*\*

## 3. 有序性

**&gt; synchronized**

synchronized语义表示锁在同一时刻只能由一个线程进行获取，当锁被占用后，其他线程只能等待。因此，synchronized语义就要求线程在访问读写共享变量时只能“串行”执行，因此\*\***synchronized具有有序性**\*\*。

**&gt; volatile**

在java内存模型中说过，为了性能优化，编译器和处理器会进行指令重排序；也就是说java程序天然的有序性可以总结为：\*\***如果在本线程内观察，所有的操作都是有序的；如果在一个线程观察另一个线程，所有的操作都是无序的**\*\*。在单例模式的实现上有一种双重检验锁定的方式（Double-checked Locking）。代码如下：

```
public class Singleton {
        private Singleton() { }
        private volatile static Singleton instance;
        public Singleton getInstance(){
            if(instance==null){
                synchronized (Singleton.class){
                    if(instance==null){
                        instance = new Singleton();
                    }
                }
            }
            return instance;
        }
    }
```

这里为什么要加volatile了？我们先来分析一下不加volatile的情况，有问题的语句是这条：

&gt; instance = new Singleton\(\);

这条语句实际上包含了三个操作：1.分配对象的内存空间；2.初始化对象；3.设置instance指向刚分配的内存地址。但由于存在重排序的问

题，可能有以下的执行顺序：

![](/assets/不加volatile可能的执行时序.png)

!\[不加volatile可能的执行时序\]\([http://upload-images.jianshu.io/upload\_images/2615789-e7931260b0449eb1.png?imageMogr2/auto-](http://upload-images.jianshu.io/upload_images/2615789-e7931260b0449eb1.png?imageMogr2/auto-)orient/strip%7CimageView2/2/w/1240\)

如果2和3进行了重排序的话，线程B进行判断if\(instance==null\)时就会为true，而实际上这个instance并没有初始化成功，显而易见对线程B来说之后的操作就会是错得。而\*\***用volatile修饰\*\*的话就可以禁止2和3操作重排序，从而避免这种情况。\*\*volatile包含禁止指令重排序的语义，其具有有序性**\*\*。

## 4. 可见性

**可见性是指当一个线程修改了共享变量后，其他线程能够立即得知这个修改**。通过之前对\[synchronzed\]

\([https://juejin.im/post/5ae6dc04f265da0ba351d3ff\)内存语义进行了分析，当线程获取锁时会从主内存中获取共享变量的最新值，释放锁](https://juejin.im/post/5ae6dc04f265da0ba351d3ff%29内存语义进行了分析，当线程获取锁时会从主内存中获取共享变量的最新值，释放锁)

的时候会将共享变量同步到主内存中。从而，\*\***synchronized具有可见性**\*\*。同样的在\[volatile分析中\]

\([https://juejin.im/post/5ae9b41b518825670b33e6c4\)，会通过在指令中添加\*\*lock指令\*\*，以实现内存可见性。因此](https://juejin.im/post/5ae9b41b518825670b33e6c4%29，会通过在指令中添加**lock指令**，以实现内存可见性。因此), \*\***volatile具有可见性**\*\*

## 5. 总结

通过这篇文章，主要是比较了synchronized和volatile在三条性质：原子性，可见性，以及有序性的情况，归纳如下：

&gt; \*\***synchronized: 具有原子性，有序性和可见性**\*\*；

&gt; \*\***volatile：具有有序性和可见性**\*\*

# 参考文献

《java并发编程的艺术》

《深入理解java虚拟机》

