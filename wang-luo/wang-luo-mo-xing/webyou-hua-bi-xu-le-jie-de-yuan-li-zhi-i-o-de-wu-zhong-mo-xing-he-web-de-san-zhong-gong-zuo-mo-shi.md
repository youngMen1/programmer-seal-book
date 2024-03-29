# web优化必须了解的原理之I/O的五种模型和web的三种工作模式

## 图解五种I/O模型

## 图解web支持的三种工作模式

---

## 五种I/O模型：

1）阻塞I/0

2）非阻塞I/O

3）I/O复用

4）事件\(信号\)驱动I/O

5）异步I/O

为什么要发起系统调用？

```
因为进程想要获取磁盘中的数据，而能和硬件打交道的只能是内核，进程通知内核说我要磁盘中的数据，此过程就是系统调用。
```

一次I/O的完成的步骤

```
当进程发起系统调用时，这个系统调用就进入内核模式，然后开始I/O操作
```

I/O操作分为两个步骤；

```
1、磁盘把数据装载到内核的内存空间，

2、内核的内存空间的数据copy到用户的内存空间中(此过程是I/O发生的地方)
```

以下是进程获取数据的详细图解过程；  
![img](/static/image/205126317.png)  
整个过程：此进程需要对磁盘中的数据进行操作，则会向内核发起一个系统调用，然后此进程，将会被切换出去，此进程会被挂起或者进入睡眠状态，也叫不可中断的睡眠，因为数据还没有得到，只有等到系统调用的结果完成后，则进程会被唤醒，继续接下来的操作，从系统调用的开始到系统调用结束经过的步骤：

①进程向内核发起一个系统调用，

②内核接收到系统调用，知道是对文件的请求，于是告诉磁盘，把文件读取出来

③磁盘接收到来着内核的命令后，把文件载入到内核的内存空间里面

④内核的内存空间接收到数据之后，把数据copy到用户进程的内存空间\(此过程是I/O发生的地方\)

⑤进程内存空间得到数据后，给内核发送通知

⑥内核把接收到的通知回复给进程，此过程为唤醒进程，然后进程得到数据，进行下一步操作  
I/O发生的地方才会出现阻塞或非阻塞

## 阻塞：进程发起I/O调用，进程又不得不等待I/O的完成，此时CPU把进程切换出去，进程处于睡眠状态则此过程为阻塞I/O

阻塞I/O系统怎么通知进程？  
I/O完成，系统直接通知进程，则进程被唤醒

### 阻塞I/O的图解:

![img](/static/image/205500239.png)

## 非阻塞：进程发起I/O调用，I/O自己知道需过一段时间完成，就立即通知进程进行别的操作，则为非阻塞I/O

非阻塞I/O，系统怎么通知进程？

每隔一段时间，问内核数据是否准备完成，系统完成后，则进程获取数据，继续执行\(此过程也称盲等待\)

### 非阻塞I/O的图解：

![img](/static/image/205605819.png)

## I/O复用的图解：  

![img](/static/image/205635176.png)  

## 事件\(信号\)驱动I/O的图解：

水平触发的事件驱动机制；内核通知进程来读取数据，进程没来读取数据，内核需要一次一次的通知进程；
边缘触发的事件驱动机制；内核只通知一次让进程来读取数据，进程可以在超时时间之内随时来读取数据。
nginx就采用了边缘触发的事件驱动机制，这就是为什么nginx的并发性比apache好，当然nginx的性能比apache好，还有其它方面，如nginx支持异步I/O，mmap(内存映射)等等

![img](/static/image/210003879.png)

## 异步I/O的图解：  
![img](/static/image/210054915.png)  
前四种I/O属于同步操作，最后的一种则属于异步操作  
五种I/O模型的比较：  
![img](/static/image/212627938.png)  
# web的三种工作模式

## Prefork工作原理

```
主进程生成多个工作进程，由工作进程一对一的去响应客户端的请求
```

图解Prefork工作原理：  
  ![img](/static/image/084450144.png)  
## Worker工作原理

```
主进程生成多个工作进程，每个工作进程生成一个多个线程，每个线程去
```

响应客户端的请求

图解Worker工作原理：

![img](/static/image/084552193.png)

## Event工作原理

```
主进程生成多个工作进程，每个工程进程响应多个客户端的请求，当接收
```

到客户端的I/O操作请求后，把I/O操作交给内核执行，进程去响应其他客

户端的请求，此进程最后接到内核的通知，然后通过此进程回复客户端的

请求结果，通过事件回调函数

图解Event工作原理：  
  ![img](/static/image/084608255.png)

