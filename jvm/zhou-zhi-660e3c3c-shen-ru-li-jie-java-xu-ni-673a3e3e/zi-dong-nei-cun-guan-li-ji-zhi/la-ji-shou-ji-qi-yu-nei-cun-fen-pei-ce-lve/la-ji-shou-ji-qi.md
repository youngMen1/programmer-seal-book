# 1.基本介绍

#### 1.Serial收集器

#### 2.ParNew收集器

#### 3.Parallel Scavenge收集器

#### 4.Serial Old收集器

#### 5.Parallel Old收集器

#### 6.CMS收集器

#### 7.G1收集器

## 1.1.Serial收集器

这个收集器是一个单线程的收集器，在它进行垃圾收集时，必须暂停其他所有的工作线程，直到它线程结束。

### 1.1.1.特点

主要针对新生代；

采用复制算法；

单线程收集；

进行垃圾收集时，必须暂停所有工作线程，直到完成；

即会"Stop The World"；

Serial/Serial Old组合收集器运行示意图如下：

![](/static/image/20180611160921828.png)

### 1.1.2.应用场景

依然是HotSpot在Client模式下默认的新生代收集器；

也有优于其他收集器的地方：

简单高效（与其他收集器的单线程相比）；

对于限定单个CPU的环境来说，Serial收集器没有线程交互（切换）开销，可以获得最高的单线程收集效率；

在**用户的桌面应用场景中**，可用内存一般不大（几十M至一两百M），可以在较短时间内完成垃圾收集（几十MS至一百多MS）,只要不频繁发生，这是可以接受的

**1、设置参数**

"-XX:+UseSerialGC"：添加该参数来显式的使用串行垃圾收集器；

**2、Stop TheWorld说明**

JVM在后台自动发起和自动完成的，在用户不可见的情况下，把用户正常的工作线程全部停掉，即GC停顿；

会带给用户不良的体验；

从JDK1.3到现在，从Serial收集器-》Parallel收集器-》CMS-》G1，用户线程停顿时间不断缩短，但仍然无法完全消除；

## 1.2.ParNew收集器

ParNew垃圾收集器是Serial收集器的多线程版本。

### 1.2.1.特点

除了多线程外，其余的行为、特点和Serial收集器一样；  
如Serial收集器可用控制参数、收集算法、Stop The World、内存分配规则、回收策略等；  
两个收集器共用了不少代码；  
ParNew/Serial Old组合收集器运行示意图如下：  
![](/static/image/2018061116094429.png)  
**1.应用场景**  
在Server模式下，ParNew收集器是一个非常重要的收集器，因为除Serial外，目前只有它能与CMS收集器配合工作；  
但在单个CPU环境中，不会比Serail收集器有更好的效果，因为存在线程交互开销。  
**2.设置参数**  
"-XX:+UseConcMarkSweepGC"：指定使用CMS后，会默认使用ParNew作为新生代收集器；  
"-XX:+UseParNewGC"：强制指定使用ParNew；  
"-XX:ParallelGCThreads"：指定垃圾收集的线程数量，ParNew默认开启的收集线程与CPU的数量相同；  
**3.为什么只有ParNew能与CMS收集器配合**  
CMS是HotSpot在JDK1.5推出的第一款真正意义上的并发（Concurrent）收集器，第一次实现了让垃圾收集线程与用户线程（基本上）同时工作；  
CMS作为老年代收集器，但却无法与JDK1.4已经存在的新生代收集器Parallel Scavenge配合工作；  
因为Parallel Scavenge（以及G1）都没有使用传统的GC收集器代码框架，而另外独立实现；而其余几种收集器则共用了部分的框架代码；

## 1.3.Parallel Scavenge收集器

Parallel Scavenge收集器是一个**新生代收集器**，使用**复制算法**的**并行收集器**Parallel Scavenge 收集器使用两个参数控制吞吐量

```
MaxGCPauseMillis:控制最大的垃圾收集停顿时间
GCRatio：直接设置吞吐量的大小
```

## 

## 1.4.Serial Old收集器

## 1.5.Parallel Old收集器

## 1.6.CMS收集器

## 1.7.G1收集器



