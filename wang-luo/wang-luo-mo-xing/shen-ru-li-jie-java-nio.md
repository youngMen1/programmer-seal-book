**初识NIO：**


在 JDK 1. 4 中 新 加入 了 NIO\( New Input/ Output\) 类, 引入了一种基于通道和缓冲区的 I/O 方式，它可以使用 Native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆的 DirectByteBuffer 对象作为这块内存的引用进行操作，避免了在 Java 堆和 Native 堆中来回复制数据。

NIO 是一种同步非阻塞的 IO 模型。同步是指线程不断轮询 IO 事件是否就绪，非阻塞是指线程在等待 IO 的时候，可以同时做其他任务。同步的核心就是 Selector，Selector 代替了线程本身轮询 IO 事件，避免了阻塞同时减少了不必要的线程消耗；非阻塞的核心就是通道和缓冲区，当 IO 事件就绪时，可以通过写道缓冲区，保证 IO 的成功，而无需线程阻塞式地等待。


**Buffer：**


为什么说NIO是基于缓冲区的IO方式呢？因为，当一个链接建立完成后，IO的数据未必会马上到达，为了当数据到达时能够正确完成IO操作，在BIO（阻塞IO）中，等待IO的线程必须被阻塞，以全天候地执行IO操作。为了解决这种IO方式低效的问题，引入了缓冲区的概念，当数据到达时，可以预先被写入缓冲区，再由缓冲区交给线程，因此线程无需阻塞地等待IO。


**通道：**

当执行：SocketChannel.write\(Buffer\)，便将一个 buffer 写到了一个通道中。如果说缓冲区还好理解，通道相对来说就更加抽象。网上博客难免有写不严谨的地方，容易使初学者感到难以理解。

引用 Java NIO 中权威的说法：通道是 I/O 传输发生时通过的入口，而缓冲区是这些数 据传输的来源或目标。对于离开缓冲区的传输，您想传递出去的数据被置于一个缓冲区，被传送到通道。对于传回缓冲区的传输，一个通道将数据放置在您所提供的缓冲区中。

例如 有一个服务器通道 ServerSocketChannel serverChannel，一个客户端通道 SocketChannel clientChannel；服务器缓冲区：serverBuffer，客户端缓冲区：clientBuffer。

当服务器想向客户端发送数据时，需要调用：clientChannel.write\(serverBuffer\)。当客户端要读时，调用 clientChannel.read\(clientBuffer\)

当客户端想向服务器发送数据时，需要调用：serverChannel.write\(clientBuffer\)。当服务器要读时，调用 serverChannel.read\(serverBuffer\)

这样，通道和缓冲区的关系似乎更好理解了。在实践中，未必会出现这种双向连接的蠢事（然而这确实存在的，后面的内容还会涉及），但是可以理解为在NIO中：如果想将Data发到目标端，则需要将存储该Data的Buffer，写入到目标端的Channel中，然后再从Channel中读取数据到目标端的Buffer中。


**Selector：**

通道和缓冲区的机制，使得线程无需阻塞地等待IO事件的就绪，但是总是要有人来监管这些IO事件。这个工作就交给了selector来完成，这就是所谓的同步。 

Selector允许单线程处理多个 Channel。如果你的应用打开了多个连接（通道），但每个连接的流量都很低，使用Selector就会很方便。

要使用Selector，得向Selector注册Channel，然后调用它的select\(\)方法。这个方法会一直阻塞到某个注册的通道有事件就绪，这就是所说的轮询。一旦这个方法返回，线程就可以处理这些事件。

Selector中注册的感兴趣事件有：

* OP\_ACCEPT

* OP\_CONNECT

* OP\_READ

* OP\_WRITE

**优化：**

一种优化方式是：将Selector进一步分解为Reactor，将不同的感兴趣事件分开，每一个Reactor只负责一种感兴趣的事件。这样做的好处是：1、分离阻塞级别，减少了轮询的时间；2、线程无需遍历set以找到自己感兴趣的事件，因为得到的set中仅包含自己感兴趣的事件。




