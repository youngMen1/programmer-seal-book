# 1.基本介绍

![](/static/image/20191125190657919.png)  
后台微服务架构中的一个订单系统，一般几百万的访问量基本上就是集中在3-4个小时内完成，平均下来每秒100单左右。我们部署三台服务器同时提供服务，平均每台服务器每秒钟处理30单左右。一个订单假设会产生1KB的对象（估计方法：这个对象内包含哪些声明的变量，在这个对象内会嵌套哪些对象挖到最内层计算变量所占的字节数，从而来预估每个对象所占得内存大小），每秒钟就会产生30KB的订单对象。当然，下单时还会涉及其他对象，如购物车、库存、优惠券、积分等等，因此每秒钟产生对象所占内存我们扩大30倍，约等于每秒1MB对象产生。对于实际的生产环境中，我们要根据系统的核心业务去进行对象大小的估算。公司一般采用双核4G或4核8G的机器，针对这两种不同配置的机器，我们来做JVM参数的分析。

## 1.1.双核4G机器

由于操作系统的运行本身也会消耗内存，那么分配给我们JVM进程的内存约为2G。再去除方法区和线程栈，**分配给我们堆内存的容量预计也就1G左右。**年轻代内存大小300M、老年代700M，若每秒产生1MB的对象，预计20-30秒的时间，年轻代就会被放满，从而去**触发minor GC（新生代垃圾回收）**比较频繁。

## 1.2.四核8G机器

JVM进程就会分配4-5G的内存空间，堆内存会分配到3-4G，于是年轻代至少分配2G，这样至少半个小时才会把年轻代放满而触发minor GC，因此线上服务器多用4核8G的配置minor GC和full GC都有Stop The World时间，而minor GC对系统影响不是很大。  
若访问量一直是这个水平采用4G内存也没问题，但考虑到电商会经常搞大促活动，访问量会在短时间内激增。假设访问量增加20倍，那么三台服务器每台每秒就会处理600单的订单数据，每秒产生20MB的对象，而如果仍然采用4G内存，预计10秒左右就会将年轻代放满。当然，如果每秒需要处理600单订单，考虑到实际的生产环境，网络带宽、磁盘IO、数据库、线程大量切换等系统资源都会产生系统瓶颈。因此每秒钟并不一定就会处理600单订单，那么就要引出我们在**《JVM虚拟机运行时数据区》**博客末尾提到的两个问题：

### 1.2.1.问题

**1.什么样的对象可能被挪到老年代被fullgc回收掉？**

由于系统性能达到瓶颈，有些订单对象会在默认值15次minor GC（新生代）后仍未被释放，因此就会被移到老年代，这种恶性运行状况持续下去就会触发我们最不愿见到的full GC（老年代）。

**2.针对上钟情况如何做jvm参数调优？**

如果采用8G内存，年轻代会分配2G内存，需要约2分钟才会将内存空间放满触发minor GC，而这段时间内足以将订单对象处理完就不会有对象被频繁的移动到老年代。具体参数可以进行如下设置：eden：1.6GB s0：200MB s1：200MB old：1G

![](/static/image/20191125195431647.png)

根据每个微服务的核心业务，每秒产生对象大小和服务器内存情况以及预期的GC间隔时间去设定各个区域的内存大小。

## 1.3.逃逸分析

**JVM三种运行模式**

**解释模式**：只使用解释器，执行一行jvm字节码就编译一行为机器码。

**编译模式**：只使用编译器，将所有jvm字节码一次编译为机器码，然后一次性执行所有机器码。

**混合模式**：依然使用解释器执行代码，但对于一些“热点代码”会采用编译模式执行，JVM一般采用混合模式执行代码。

解释模式启动快，对于只需要执行部分代码，并且大多数代码只会执行一次的情况比较适合；

编译模式启动慢，但是后期执行速度快，而且比较占用内存，因为机器码的数量至少是JVM字节码的十倍以上，这种模式适合代码可能会被反复执行的场景；

混合模式是JVM默认采用的执行代码方式，一开始还是解释执行，但是对于少部分 “热点 ”代码会采用编译模式执行，这些热点代码对应的机器码会被缓存起来，下次再执行无需再编译，这就是我们常见的JIT\(Just In Time Compiler\)即时编译技术。 在即时编译过程中JVM可能会对我们的代码做一些优化，比如对象逃逸分析等。

**编译体系**

在Java的编译体系中，一个Java的源代码文件变成计算机可执行的机器指令的过程中，需要经过两段编译，第一段是把.java文件转换成.class文件（java文件---》字节码）。第二段编译是把.class转换成机器指令的过程（字节码---》机器码）。

第一段编译就是javac命令。

在第二编译阶段，JVM 通过解释字节码将其翻译成对应的机器指令，逐条读入，逐条解释翻译。很显然，经过解释执行，其执行速度必然会比可执行的二进制字节码程序慢很多。这就是传统的JVM的解释器（Interpreter）的功能。为了解决这种效率问题，引入了 JIT（即时编译） 技术。

引入了 JIT 技术后，Java程序还是通过解释器进行解释执行，当JVM发现某个方法或代码块运行特别频繁的时候，就会认为这是“热点代码”（Hot Spot Code\)。然后JIT会把部分“热点代码”翻译成本地机器相关的机器码，并进行优化，然后再把翻译后的机器码缓存起来，以备下次使用。

由于关于JIT编译和热点检测的内容，我在深入分析Java的编译原理中已经介绍过了，这里就不在赘述，本文主要来介绍下JIT中的优化。JIT优化中最重要的一个就是逃逸分析

**逃逸分析的基本行为就是分析对象动态作用域：**当一个对象在方法中被定义后，它可能被外部方法所引用，例如作为调用参数传递到其他地方中，称为**方法逃逸**

```
public static StringBuffer craeteStringBuffer(String s1, String s2) {
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    return sb;
}

public static String createStringBuffer(String s1, String s2) {
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    return sb.toString();
}
```

第一段代码中的sb就逃逸了，而第二段代码中的sb就没有逃逸。

**使用逃逸分析，编译器可以对代码做如下优化：**

将堆分配转化为栈分配。如果一个对象在子程序中被分配，要使指向该对象的指针永远不会逃逸，对象可能是栈分配的候选，而不是堆分配。

# 2.参考

日均百万级商城如何JVM性能调优：

[https://blog.csdn.net/Im\_javaer/article/details/103243699](https://blog.csdn.net/Im_javaer/article/details/103243699)

