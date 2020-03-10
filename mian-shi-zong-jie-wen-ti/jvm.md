## 1、JVM堆的基本结构。 {#1、JVM堆的基本结构。}

在JVM中堆空间划分如下图所示

![img](/static/image/JVM堆.png)  
上图中，刻画了Java程序运行时的堆空间,可以简述成如下2条

1、JVM中堆空间可以分成三个大区，新生代、老年代、永久代

2、新生代可以划分为三个区，Eden区，两个Survivor区，在HotSpot虚拟机Eden和Survivor的大小比例为8:1

## 2、JVM的垃圾算法有哪几种？CMS垃圾回收的基本流程？ {#2、JVM的垃圾算法有哪几种？CMS垃圾回收的基本流程？}

四种：标记-清除算法、复制算法、标记-整理算法、分代收集算法

垃圾收集器有七种：Serial、ParNew、Parallel Scavenge、CMS、Serial Old、Parrallel Old、G1

CMS全称为Concurrent Mark Sweep，是一款并发、使用标记-清除算法的gc收集器。

总体来说CMS的执行过程可以分为以下几个阶段：  
初始标记 -&gt; 并发标记 -&gt; 重新标记 -&gt; 并发清理 -&gt; 重置

