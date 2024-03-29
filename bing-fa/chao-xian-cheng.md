# 1.超线程技术

每个单位时间内，CPU只能处理一个线程（Thread）。除非有两个核心处理单元，否则要想在单位时间内处理超过一个的线程是不可能的。

超线程HT（Hyper-Threading）技术是在单个核心处理单元中集成两个逻辑处理单元，也就是一个实体内核（共享的运算单元），两个逻辑内核（有各自独立的处理器状态），在一颗CPU同时执行多个程序而共同分享一颗CPU内的资源，理论上要像两颗CPU一样在同一时间执行两个线程，P4处理器需要多加入一个Logical CPU Pointer（逻辑处理单元）。因此新一代的P4 的面积比以往的P4增大了5%。而其余部分如ALU（整数运算单元）、FPU（浮点运算单元）、L2 Cache（二级缓存）则保持不变，这些部分是被分享的。

![](/static/image/76-1F11G01321.jpg)

虽然采用**超线程技术能同时执行两个线程**，但它并不象两个真正的CPU那样，每各CPU都具有独立的资源。当两个线程都同时需要某一个资源时，其中一个要暂时停止，并让出资源，直到这些资源闲置后才能继续。因此超线程的性能并不等于两颗CPU的性能。

# 2.多处理器

多处理器（Multiprocessor）系统由不同芯片上的多个处理器组成。多处理器系统因IT服务器的应用在上世纪九十年代得以普及。在当时，它们是可以插入机架服务器的处理器主板。现在，多处理器系统可以构建在同一块电路板上，处理器之间通过一个高速通信接口连接。

![](/static/image/76-1F11G01321-51.jpg)

多处理器系统的复杂度低于多核系统，因为它们本质是互连在一起的单芯片CPU。多处理器系统的不足在于其高昂的价格，因为它们需要多个芯片，这比单芯片解决方案要昂贵得多。

# 3.双核与多核处理器

双核处理器是指单个芯片上有两个CPU，而多核处理器则是指在单个芯片上包含任意多个（如2、4或8）CPU的处理器。多核处理器的挑战在于软件开发部分。系统性能提升的多少直接与通过多线程编程源代码的并行程度有关。

![](/static/image/76-1F11G01322.jpg)

多核处理器共享具有短程互联结构的高速缓存和MMU内存管理单元

# 4.总结

超线程由于处理器实际上只有一个核心，能够提升的效能约为5~15%左右，且万一发生资源互抢的情形时，整体效能反而会下降。双核共用Cache，程序设计合理性能可能比双处理器性能更好，多处理器可能还需在两个Cache间传输数据，多核和超线程的区别如图

![](/static/image/76-1F11G01322-50.jpg)

上文便是超线程、多核、多处理器的区别和特点介绍，用户在进行单任务操作时候，没有必要打开超线程，只有多任务操作时候可以适时打开超线程。

