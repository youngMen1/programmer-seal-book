## 1、JVM堆的基本结构。 {#1、JVM堆的基本结构。}

在JVM中堆空间划分如下图所示

![img](/static/image/JVM堆.png)  
上图中，刻画了Java程序运行时的堆空间,可以简述成如下2条

1、JVM中堆空间可以分成三个大区，新生代、老年代、永久代

2、新生代可以划分为三个区，Eden区，两个Survivor区，在HotSpot虚拟机Eden和Survivor的大小比例为8:1

## 、JVM的垃圾算法有哪几种？CMS垃圾回收的基本流程？ {#2、JVM的垃圾算法有哪几种？CMS垃圾回收的基本流程？}



