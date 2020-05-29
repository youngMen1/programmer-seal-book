# 面试问题整理
### JVM常见参数

1. -Xms20M：表示设置JVM启动内存的最小值为20M，必须以M为单位
2. -Xmx20M：表示设置JVM启动内存的最大值为20M，必须以M为单位。将-Xmx和-Xms设置为一样可以避免JVM内存自动扩展。大的项目-Xmx和-Xms一般都要设置到10G、20G甚至还要高
3. -verbose:gc：表示输出虚拟机中GC的详细情况
4. -Xss128k：表示可以设置虚拟机栈的大小为128k
5. -Xoss128k：表示设置本地方法栈的大小为128k。不过HotSpot并不区分虚拟机栈和本地方法栈，因此对于HotSpot来说这个参数是无效的
6. -XX:PermSize=10M：表示JVM初始分配的永久代（方法区）的容量，必须以M为单位
7. -XX:MaxPermSize=10M：表示JVM允许分配的永久代（方法区）的最大容量，必须以M为单位，大部分情况下这个参数默认为64M
8. -Xnoclassgc：表示关闭JVM对类的垃圾回收
9. -XX:+TraceClassLoading表示查看类的加载信息
10. -XX:+TraceClassUnLoading：表示查看类的卸载信息
11. -XX:NewRatio=4：表示设置年轻代（包括Eden和两个Survivor区）/老年代 的大小比值为1：4，这意味着年轻代占整个堆的1/5
12. -XX:SurvivorRatio=8：表示设置2个Survivor区：1个Eden区的大小比值为2:8，这意味着Survivor区占整个年轻代的1/5，这个参数默认为8
13. -Xmn20M：表示设置年轻代的大小为20M
14. -XX:+HeapDumpOnOutOfMemoryError：表示可以让虚拟机在出现内存溢出异常时Dump出当前的堆内存转储快照
15. -XX:+UseG1GC：表示让JVM使用G1垃圾收集器
16. -XX:+PrintGCDetails：表示在控制台上打印出GC具体细节
17. -XX:+PrintGC：表示在控制台上打印出GC信息
18. -XX:PretenureSizeThreshold=3145728：表示对象大于3145728（3M）时直接进入老年代分配，这里只能以字节作为单位
19. -XX:MaxTenuringThreshold=1：表示对象年龄大于1，自动进入老年代,如果设置为0的话，则年轻代对象不经过Survivor区，直接进入年老代。对于年老代比较多的应用，可以提高效率。如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象在年轻代的存活时间，增加在年轻代被回收的概率。
20. -XX:CompileThreshold=1000：表示一个方法被调用1000次之后，会被认为是热点代码，并触发即时编译
21. -XX:+PrintHeapAtGC：表示可以看到每次GC前后堆内存布局
22. -XX:+PrintTLAB：表示可以看到TLAB的使用情况
23. -XX:+UseSpining：开启自旋锁
24. -XX:PreBlockSpin：更改自旋锁的自旋次数，使用这个参数必须先开启自旋锁
25. -XX:+UseSerialGC：表示使用jvm的串行垃圾回收机制，该机制适用于单核cpu的环境下
26. -XX:+UseParallelGC：表示使用jvm的并行垃圾回收机制，该机制适合用于多cpu机制，同时对响应时间无强硬要求的环境下，使用-XX:ParallelGCThreads=<N>设置并行垃圾回收的线程数，此值可以设置与机器处理器数量相等。
27. -XX:+UseParallelOldGC：表示年老代使用并行的垃圾回收机制
28. -XX:+UseConcMarkSweepGC：表示使用并发模式的垃圾回收机制，该模式适用于对响应时间要求高，具有多cpu的环境下
29. -XX:MaxGCPauseMillis=100：设置每次年轻代垃圾回收的最长时间，如果无法满足此时间，JVM会自动调整年轻代大小，以满足此值。
30. -XX:+UseAdaptiveSizePolicy：设置此选项后，并行收集器会自动选择年轻代区大小和相应的Survivor区比例，以达到目标系统规定的最低响应时间或者收集频率等，此值建议使用并行收集器时，一直打开

### JVM调优目标-何时需要做jvm调优

1. heap 内存（老年代）持续上涨达到设置的最大内存值；
2. Full GC 次数频繁；
3. GC 停顿时间过长（超过1秒）；
4. 应用出现OutOfMemory 等内存异常；
5. 应用中有使用本地缓存且占用大量内存空间；
6. 系统吞吐量与响应性能不高或下降。

### JVM调优实战

1. Major GC和Minor GC频繁

    首先优化Minor GC频繁问题。通常情况下，由于新生代空间较小，Eden区很快被填满，就会导致频繁Minor GC，因此可以通过增大新生代空间来降低Minor GC的频率。例如在相同的内存分配率的前提下，新生代中的Eden区增加一倍，Minor GC的次数就会减少一半。
    
    扩容Eden区虽然可以减少Minor GC的次数，但会增加单次Minor GC时间么？扩容后，Minor GC时增加了T1（扫描时间），但省去T2（复制对象）的时间，更重要的是对于虚拟机来说，复制对象的成本要远高于扫描成本，所以，单次Minor GC时间更多取决于GC后存活对象的数量，而非Eden区的大小。因此如果堆中短期对象很多，那么扩容新生代，单次Minor GC时间不会显著增加。
    
2. 请求高峰期发生GC，导致服务可用性下降

    由于跨代引用的存在，CMS在Remark阶段必须扫描整个堆，同时为了避免扫描时新生代有很多对象，增加了可中断的预清理阶段用来等待Minor GC的发生。只是该阶段有时间限制，如果超时等不到Minor GC，Remark时新生代仍然有很多对象，我们的调优策略是，通过参数强制Remark前进行一次Minor GC，从而降低Remark阶段的时间。
    另外，类似的JVM是如何避免Minor GC时扫描全堆的？ 经过统计信息显示，老年代持有新生代对象引用的情况不足1%，根据这一特性JVM引入了卡表（card table）来实现这一目的。卡表的具体策略是将老年代的空间分成大小为512B的若干张卡（card）。卡表本身是单字节数组，数组中的每个元素对应着一张卡，当发生老年代引用新生代时，虚拟机将该卡对应的卡表元素设置为适当的值。如上图所示，卡表3被标记为脏（卡表还有另外的作用，标识并发标记阶段哪些块被修改过），之后Minor GC时通过扫描卡表就可以很快的识别哪些卡中存在老年代指向新生代的引用。这样虚拟机通过空间换时间的方式，避免了全堆扫描。

3. STW过长的GC

    对于性能要求很高的服务，建议将MaxPermSize和MinPermSize设置成一致（JDK8开始，Perm区完全消失，转而使用元空间。而元空间是直接存在内存中，不在JVM中），Xms和Xmx也设置为相同，这样可以减少内存自动扩容和收缩带来的性能损失。虚拟机启动的时候就会把参数中所设定的内存全部化为私有，即使扩容前有一部分内存不会被用户代码用到，这部分内存在虚拟机中被标识为虚拟内存，也不会交给其他进程使用。

4. 外部命令导致系统缓慢

    一个数字校园应用系统，发现请求响应时间比较慢，通过操作系统的mpstat工具发现CPU使用率很高，并且系统占用绝大多数的CPU资 源的程序并不是应用系统本身。每个用户请求的处理都需要执行一个外部shell脚本来获得系统的一些信息，执行这个shell脚本是通过Java的 Runtime.getRuntime().exec()方法来调用的。这种调用方式可以达到目的，但是它在Java 虚拟机中是非常消耗资源的操作，即使外部命令本身能很快执行完毕，频繁调用时创建进程 的开销也非常可观。Java虚拟机执行这个命令的过程是:首先克隆一个和当前虚拟机拥有一 样环境变量的进程，再用这个新的进程去执行外部命令，最后再退出这个进程。如果频繁执 行这个操作，系统的消耗会很大，不仅是CPU，内存负担也很重。用户根据建议去掉这个Shell脚本执行的语句，改为使用Java的API去获取这些信息后， 系统很快恢复了正常。
    
5. 由Windows虚拟内存导致的长时间停顿

    一个带心跳检测功能的GUI桌面程序，每15秒会发送一次心跳检测信号，如果 对方30秒以内都没有信号返回，那就认为和对方程序的连接已经断开。程序上线后发现心跳 检测有误报的概率，查询日志发现误报的原因是程序会偶尔出现间隔约一分钟左右的时间完 全无日志输出，处于停顿状态。
    
    因为是桌面程序，所需的内存并不大(-Xmx256m)，所以开始并没有想到是GC导致的 程序停顿，但是加入参数-XX:+PrintGCApplicationStoppedTime-XX:+PrintGCDateStamps- Xloggc:gclog.log后，从GC日志文件中确认了停顿确实是由GC导致的，大部分GC时间都控 制在100毫秒以内，但偶尔就会出现一次接近1分钟的GC。

    从GC日志中找到长时间停顿的具体日志信息(添加了-XX:+PrintReferenceGC参数)， 找到的日志片段如下所示。从日志中可以看出，真正执行GC动作的时间不是很长，但从准 备开始GC，到真正开始GC之间所消耗的时间却占了绝大部分。
    
    除GC日志之外，还观察到这个GUI程序内存变化的一个特点，当它最小化的时候，资源 管理中显示的占用内存大幅度减小，但是虚拟内存则没有变化，因此怀疑程序在最小化时它 的工作内存被自动交换到磁盘的页面文件之中了，这样发生GC时就有可能因为恢复页面文 件的操作而导致不正常的GC停顿。在Java的GUI程序中要避免这种现象，可以 加入参数“-Dsun.awt.keepWorkingSetOnMinimize=true”来解决。

## Java基础

### HashMap和ConcurrentHashMap

由于HashMap是线程不同步的，虽然处理数据的效率高，但是在多线程的情况下存在着安全问题，因此设计了CurrentHashMap来解决多线程安全问题。

HashMap在put的时候，插入的元素超过了容量（由负载因子决定）的范围就会触发扩容操作，就是rehash，这个会重新将原数组的内容重新hash到新的扩容数组中，在多线程的环境下，存在同时其他的元素也在进行put操作，如果hash值相同，可能出现同时在同一数组下用链表表示，造成闭环，导致在get时会出现死循环，所以HashMap是线程不安全的。

HashMap的环：若当前线程此时获得ertry节点，但是被线程中断无法继续执行，此时线程二进入transfer函数，并把函数顺利执行，此时新表中的某个位置有了节点，之后线程一获得执行权继续执行，因为并发transfer，所以两者都是扩容的同一个链表，当线程一执行到e.next = new table[i] 的时候，由于线程二之前数据迁移的原因导致此时new table[i] 上就有ertry存在，所以线程一执行的时候，会将next节点，设置为自己，导致自己互相使用next引用对方，因此产生链表，导致死循环。

在JDK1.7版本中，ConcurrentHashMap维护了一个Segment数组，Segment这个类继承了重入锁ReentrantLock，并且该类里面维护了一个 HashEntry<K,V>[] table数组，在写操作put，remove，扩容的时候，会对Segment加锁，所以仅仅影响这个Segment，不同的Segment还是可以并发的，所以解决了线程的安全问题，同时又采用了分段锁也提升了并发的效率。在JDK1.8版本中，ConcurrentHashMap摒弃了Segment的概念，而是直接用Node数组+链表+红黑树的数据结构来实现，并发控制使用Synchronized和CAS来操作，整个看起来就像是优化过且线程安全的HashMap。

### HashMap如果我想要让自己的Object作为K应该怎么办

1. 重写hashCode()是因为需要计算存储数据的存储位置，需要注意不要试图从散列码计算中排除掉一个对象的关键部分来提高性能，这样虽然能更快但可能会导致更多的Hash碰撞；
2. 重写equals()方法，需要遵守自反性、对称性、传递性、一致性以及对于任何非null的引用值x，x.equals(null)必须返回false的这几个特性，目的是为了保证key在哈希表中的唯一性；

### volatile

volatile在多处理器开发中保证了共享变量的“ 可见性”。可见性的意思是当一个线程修改一个共享变量时，另外一个线程能读到这个修改的值。(共享内存，私有内存)

### Atomic类的CAS操作

CAS是英文单词CompareAndSwap的缩写，中文意思是：比较并替换。CAS需要有3个操作数：内存地址V，旧的预期值A，即将要更新的目标值B。CAS指令执行时，当且仅当内存地址V的值与预期值A相等时，将内存地址V的值修改为B，否则就什么都不做。整个比较并替换的操作是一个原子操作。如 Intel 处理器，比较并交换通过指令的 cmpxchg 系列实现。

### CAS操作ABA问题：

如果在这段期间它的值曾经被改成了B，后来又被改回为A，那CAS操作就会误认为它从来没有被改变过。Java并发包为了解决这个问题，提供了一个带有标记的原子引用类“AtomicStampedReference”，它可以通过控制变量值的版本来保证CAS的正确性。

### Synchronized和Lock的区别

1. 首先synchronized是java内置关键字在jvm层面，Lock是个java类。
2. synchronized无法判断是否获取锁的状态，Lock可以判断是否获取到锁，并且可以主动尝试去获取锁。
3. synchronized会自动释放锁(a 线程执行完同步代码会释放锁 ；b 线程执行过程中发生异常会释放锁)，Lock需在finally中手工释放锁（unlock()方法释放锁），否则容易造成线程死锁。
4. 用synchronized关键字的两个线程1和线程2，如果当前线程1获得锁，线程2线程等待。如果线程1阻塞，线程2则会一直等待下去，而Lock锁就不一定会等待下去，如果尝试获取不到锁，线程可以不用一直等待就结束了。
5. synchronized的锁可重入、不可中断、非公平，而Lock锁可重入、可判断、可公平（两者皆可）
6. Lock锁适合大量同步的代码的同步问题，synchronized锁适合代码少量的同步问题。

### AQS理论的数据结构

AQS内部有3个对象，一个是state（用于计数器，类似gc的回收计数器），一个是线程标记（当前线程是谁加锁的），一个是阻塞队列。

AQS是自旋锁，在等待唤醒的时候，经常会使用自旋的方式，不停地尝试获取锁，直到被其他线程获取成功。

AQS有两个队列，同步对列和条件队列。同步队列依赖一个双向链表来完成同步状态的管理，当前线程获取同步状态失败后，同步器会将线程构建成一个节点，并将其加入同步队列中。通过signal或signalAll将条件队列中的节点转移到同步队列。

### 如何指定多个线程的执行顺序

1. 设定一个 orderNum，每个线程执行结束之后，更新 orderNum，指明下一个要执行的线程。并且唤醒所有的等待线程。
2. 在每一个线程的开始，要 while 判断 orderNum 是否等于自己的要求值，不是，则 wait，是则执行本线程。

### 为什么要使用线程池

1. 减少创建和销毁线程的次数，每个工作线程都可以被重复利用，可执行多个任务。
2. 可以根据系统的承受能力，调整线程池中工作线程的数目，放置因为消耗过多的内存，而把服务器累趴下

### 核心线程池ThreadPoolExecutor内部参数

1. corePoolSize：指定了线程池中的线程数量
2. maximumPoolSize：指定了线程池中的最大线程数量
3. keepAliveTime：线程池维护线程所允许的空闲时间
4. unit: keepAliveTime 的单位。
5. workQueue：任务队列，被提交但尚未被执行的任务。
6. threadFactory：线程工厂，用于创建线程，一般用默认的即可。
7. handler：拒绝策略。当任务太多来不及处理，如何拒绝任务。

### 线程池的拒绝策略

1. ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。
2. ThreadPoolExecutor.DiscardPolicy：丢弃任务，但是不抛出异常。
3. ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新提交被拒绝的任务
4. ThreadPoolExecutor.CallerRunsPolicy：由调用线程（提交任务的线程）处理该任务

### 线程池的线程数量怎么确定

1. 一般来说，如果是CPU密集型应用，则线程池大小设置为N+1。
2. 一般来说，如果是IO密集型应用，则线程池大小设置为2N+1。
3. 在IO优化中，线程等待时间所占比例越高，需要越多线程，线程CPU时间所占比例越高，需要越少线程。这样的估算公式可能更适合：最佳线程数目 = （（线程等待时间+线程CPU时间）/线程CPU时间 ）* CPU数目

### 如何实现一个带优先级的线程池

利用priority参数，继承 ThreadPoolExecutor 使用 PriorityBlockingQueue 优先级队列。

### ThreadLocal的原理和实现

ThreadLoal 变量，线程局部变量，同一个 ThreadLocal 所包含的对象，在不同的 Thread 中有不同的副本。ThreadLocal 变量通常被private static修饰。当一个线程结束时，它所使用的所有 ThreadLocal 相对的实例副本都可被回收。

一个线程内可以存在多个 ThreadLocal 对象，所以其实是 ThreadLocal 内部维护了一个 Map ，这个 Map 不是直接使用的 HashMap ，而是 ThreadLocal 实现的一个叫做 ThreadLocalMap 的静态内部类。而我们使用的 get()、set() 方法其实都是调用了这个ThreadLocalMap类对应的 get()、set() 方法。

### ThreadLocal中的内存泄漏

在ThreadLocal中内存泄漏是指ThreadLocalMap中的Entry中的key为null，而value不为null。因为key为null导致value一直访问不到，而根据可达性分析导致在垃圾回收的时候进行可达性分析的时候,value可达从而不会被回收掉，但是该value永远不能被访问到，这样就存在了内存泄漏。如果 key 是强引用，那么发生 GC 时 ThreadLocalMap 还持有 ThreadLocal 的强引用，会导致 ThreadLocal 不会被回收，从而导致内存泄漏。弱引用 ThreadLocal 不会内存泄漏，对应的 value 在下一次 ThreadLocalMap 调用 set、get、remove 方法时被清除，这算是最优的解决方案。

### ThreadLocal为什么要使用弱引用和内存泄露问题

Map中的key为一个threadlocal实例. 这个Map的确使用了弱引用,不过弱引用只是针对key.每个key都弱引用指向threadlocal.假如每个key都强引用指向threadlocal，也就是上图虚线那里是个强引用，那么这个threadlocal就会因为和entry存在强引用无法被回收！造成内存泄漏 ，除非线程结束，线程被回收了，map也跟着回收。

虽然上述的弱引用解决了key，也就是线程的ThreadLocal能及时被回收，但是value却依然存在内存泄漏的问题。当把threadlocal实例置为null以后,没有任何强引用指向threadlocal实例,所以threadlocal将会被gc回收.map里面的value却没有被回收.而这块value永远不会被访问到了. 所以存在着内存泄露,因为存在一条从current thread连接过来的强引用.只有当前thread结束以后, current thread就不会存在栈中,强引用断开, Current Thread, Map, value将全部被GC回收.所以当线程的某个localThread使用完了，马上调用threadlocal的remove方法,就不会发生这种情况了。

另外其实只要这个线程对象及时被gc回收，这个内存泄露问题影响不大，但在threadLocal设为null到线程结束中间这段时间不会被回收的，就发生了我们认为的内存泄露。最要命的是线程对象不被回收的情况，这就发生了真正意义上的内存泄露。比如使用线程池的时候，线程结束是不会销毁的，会再次使用，就可能出现内存泄露。

### HashSet和HashMap

HashSet的value存的是一个static finial PRESENT = newObject()。而HashSet的remove是使用HashMap实现,则是map.remove而map的移除会返回value,如果底层value都是存null,显然将无法分辨是否移除成功。

### Boolean占几个字节

未精确定义字节。Java语言表达式所操作的boolean值，在编译之后都使用Java虚拟机中的int数据类型来代替，而boolean数组将会被编码成Java虚拟机的byte数组，每个元素boolean元素占8位。

### 阻塞非阻塞与同步异步的区别

1. 同步和异步关注的是消息通信机制，所谓同步，就是在发出一个调用时，在没有得到结果之前，该调用就不返回。但是一旦调用返回，就得到返回值了。而异步则是相反，调用在发出之后，这个调用就直接返回了，所以没有返回结果。换句话说，当一个异步过程调用发出后，调用者不会立刻得到结果。而是在调用发出后，被调用者通过状态、通知来通知调用者，或通过回调函数处理这个调用。
2. 阻塞和非阻塞关注的是程序在等待调用结果（消息，返回值）时的状态。阻塞调用是指调用结果返回之前，当前线程会被挂起。调用线程只有在得到结果之后才会返回。非阻塞调用指在不能立刻得到结果之前，该调用不会阻塞当前线程。




