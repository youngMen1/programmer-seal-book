
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
