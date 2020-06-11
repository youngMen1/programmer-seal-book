## 1、接口与抽象类的区别？

类可以实现多个接口但只能继承一个抽象类

接口里面所有的方法都是Public的，抽象类允许Private、Protected方法

JDK8前接口里面所有的方法都是抽象的且不允许有静态方法，抽象类可以有普通、静态方法，JDK8 接口可以实现默认方法和静态方法，前面加default、static关键字

## 2、Java中的异常有哪几类？分别怎么使用？ {#2、Java中的异常有哪几类？分别怎么使用？}

分为错误和异常，异常又包括运行时异常、非运行时异常

错误，如StackOverflowError、OutOfMemoryError

运行时异常，如NullPointerException、IndexOutOfBoundsException，都是RuntimeException及其子类

非运行时异常，如IOException、SQLException,都是Exception及其子类，这些异常是一定需要try catch捕获的

## 3、常用的集合类有哪些？比如List如何排序？

主要分为三类，Map、Set、List

Map: HashMap、LinkedHashMap、TreeMap

Set：HashSet、LinkedHashSet、TreeSet

List: ArrayList、LinkedList

```
Collections.sort(list);
```

## 4、ArrayList和LinkedList内部的实现大致是怎样的？他们之间的区别和优缺点？ {#4、ArrayList和LinkedList内部的实现大致是怎样的？他们之间的区别和优缺点？}

ArrayList：内部使用**数组**的形式实现了存储，利用数组的下标进行元素的访问，因此对元素的随机访问速度非常快。  
因为是数组，所以ArrayList在初始化的时候，有初始大小10，插入新元素的时候，会判断是否需要扩容，扩容的步长是0.5倍原容量，扩容方式是利用数组的复制，因此有一定的开销。

LinkedList：内部使用**双向链表的结构实现存储**，LinkedList有一个内部类作为存放元素的单元，里面有三个属性，用来存放元素本身以及前后2个单元的引用，另外LinkedList内部还有一个header属性，用来标识起始位置，LinkedList的第一个单元和最后一个单元都会指向header，因此形成了一个双向的链表结构。

ArrayList查找较快，插入、删除较慢，LinkedList查找较慢，插入、删除较快。

## 5、内存溢出是怎么回事？请举一个例子？ {#5、内存溢出是怎么回事？请举一个例子？}

内存溢出 out of memory，是指程序在申请内存时，没有足够的内存空间供其使用，出现out of memory。

内存泄漏 memory leak，是指程序在申请内存后，无法释放已申请的内存空间，一次内存泄漏危害可以忽略，但内存泄漏堆积后果很严重，无论多少内存,迟早会被占光。

```
List<Object> list = new ArrayList<>();
while (true) {
  list.add(new Object());
}
```

内存溢出可能的原因:

1、程序中存在死循环

2、静态变量和静态方法太多了

3、内存泄漏，比如说一个静态的list，一直往里放值，又因为静态变量不会被释放，所以迟早是要内存溢出的

4、大对象过多，java中的大对象是直接进入老年代的，然后当多个大对象同时工作时造成程序的可用内存非常小，比如我list中原本最多可以放1000个对象，因为可用内存太小，放了500个就放不下了。

5、还有一种很常见的情况，在把一个很大的程序直接导入，直接就内存溢出了，原因就是内存相对这个程序就是太小了，需要手动增加内存。

## 6、==和equals的区别？ {#6、-和equals的区别？}

==是运算符，而equals是Object的基本方法，==用于基本类型的数据的比较，或者是比较两个对象的引用是否相同，equals用于比较两个对象的值是否相等，例如字符串的比较

## 7、hashCode方法的作用？ {#7、hashCode方法的作用？}

1、hashCode的存在主要是用于查找的快捷性，为了配合基于散列的集合正常运行，如Hashtable，HashMap等，hashCode是用来在散列存储结构中确定对象的存储地址的；

2、如果两个对象相同，就是适用于equals\(java.lang.Object\) 方法，那么这两个对象的hashCode一定要相同；

3、如果对象的equals方法被重写，那么对象的hashCode也尽量重写，并且产生hashCode使用的对象，一定要和equals方法中使用的一致，否则就会违反上面提到的第2点；

4、两个对象的hashCode相同，并不一定表示两个对象就相同，也就是不一定适用于equals\(java.lang.Object\) 方法，只能够说明这两个对象在散列存储结构中，它们存放在同一个桶里面

## 8、NIO是什么？适用于何种场景？ {#8、NIO是什么？适用于何种场景？}

NIO是为了弥补IO操作的不足而诞生的，NIO的一些新特性有：非阻塞I/O，选择器，缓冲以及管道。管道（Channel），缓冲（Buffer） ，选择器（ Selector）是其主要特征。

如果需要管理同时打开的成千上万个连接，这些连接每次只是发送少量的数据，例如聊天服务器，这时候用NIO处理数据可能是个很好的选择。

而如果只有少量的连接，而这些连接每次要发送大量的数据，这时候传统的IO更合适。使用哪种处理数据，需要在数据的响应等待时间和检查缓冲区数据的时间上作比较来权衡选择。

[Java中NIO和IO区别和适用场景](http://www.php.cn/java-article-361228.html)

## 9、HashMap实现原理，如何保证HashMap的线程安全？ {#9、HashMap实现原理，如何保证HashMap的线程安全？}

![](/assets/hashMap put方法执行流程图.png)[java 8 Hashmap深入解析 —— put get 方法源码](https://www.cnblogs.com/jzb-blog/p/6637823.html)

HashMap是基于哈希表（链地址法）实现的，在JDK8中，当数组中链表长度大于8会转为红黑树。

```
Collections.synchronizedMap(map);
```

## 10、JVM内存结构，为什么需要GC？ {#10、JVM内存结构，为什么需要GC？}

![img](/static/image/JVM.png)

|  |
| :--- |


| 名称 | 特征 | 作用 | 配置参数 | 异常 |
| :--- | :--- | :--- | :--- | :--- |
| 程序计数器 | 占用内存小，线程私有，生命周期与线程相同 | 大致为字节码行号指示器 | 无 | 无 |
| 虚拟机栈 | 线程私有，生命周期与线程相同，使用连续的内存空间 | Java方法执行的内存模型，存储局部变量表、操作栈、动态链接、方法出口等信息 | -Xss | StackOverflowError OutOfMemoryError |
| 堆 | 线程共享，生命周期与虚拟机相同，可以不使用连续的内存地址 | 保存对象实例，所有对象（包括数组）都要在堆上分配 | -Xms -Xmx -Xmn | OutOfMemoryError |
| 方法区 | 线程共享，生命周期与虚拟机相同，可以不使用连续的内存地址 | 存储已被虚拟机加载的类信息、常量、静态变量、即时编译器后的代码等数据 | -XX:PermSize:16M -XX:MaxPermSize:64M | OutOfMemoryError |
| 运行时常量池 | 方法区的一部分，具有动态性 | 存放字面量及符号引用 | 无 | 无 |

垃圾回收可以有效的防止内存泄漏，有效的使用可以使用的内存。垃圾回收器通常是作为一个单独的低优先级的线程运行，不可预知的情况下对内存堆中已经死亡的或者长时间没有使用的对象进行清除和回收，程序员不能实时的调用垃圾回收器对某个对象或所有对象进行垃圾回收。回收机制有分代复制垃圾回收、标记垃圾回收、增量垃圾回收等方式。

## 11、NIO模型，select/epoll的区别，多路复用的原理 {#11、NIO模型，select-epoll的区别，多路复用的原理}

select，poll，epoll都是IO多路复用的机制。I/O多路复用就通过一种机制，可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作。但select，poll，epoll本质上都是同步I/O，因为他们都需要在读写事件就绪后自己负责进行读写，也就是说这个读写过程是阻塞的，而异步I/O则无需自己负责进行读写，异步I/O的实现会负责把数据从内核拷贝到用户空间。

（1）select，poll实现需要自己不断轮询所有fd集合，直到设备就绪，期间可能要睡眠和唤醒多次交替。而epoll其实也需要调用epoll\_wait不断轮询就绪链表，期间也可能多次睡眠和唤醒交替，但是它是设备就绪时，调用回调函数，把就绪fd放入就绪链表中，并唤醒在epoll\_wait中进入睡眠的进程。虽然都要睡眠和交替，但是select和poll在“醒着”的时候要遍历整个fd集合，而epoll在“醒着”的时候只要判断一下就绪链表是否为空就行了，这节省了大量的CPU时间。这就是回调机制带来的性能提升。

（2）select，poll每次调用都要把fd集合从用户态往内核态拷贝一次，并且要把current往设备等待队列中挂一次，而epoll只要一次拷贝，而且把current往等待队列上挂也只挂一次（在epoll\_wait的开始，注意这里的等待队列并不是设备等待队列，只是一个epoll内部定义的等待队列）。这也能节省不少的开销。

[select、poll、epoll之间的区别总结\[整理\]](http://www.cnblogs.com/Anker/p/3265058.html)

## 12、Java中一个字符占多少个字节，扩展再问int, long, double占多少字节 {#12、Java中一个字符占多少个字节，扩展再问int-long-double占多少字节}

1字节： byte , boolean

2字节： short , char

4字节： int , float

8字节： long , double

## 13、创建一个类的实例都有哪些办法？ {#13、创建一个类的实例都有哪些办法？}

```
Object o = new Object();
Object o = oo.clone();
Object o = Class.forName("xxx").newInstance();
```

## 14、final/finally/finalize的区别？ {#14、final-finally-finalize的区别？}

final是定义类、方法、字段的修饰符，表示类不可被继承，方法不能被重写，字段值不能被修改

finally是异常处理机制的关键字，表示最后执行

finalize是Object的一个方法，在对象被虚拟机回收时会判断是否执行该方法，当对象没有覆盖finalize方法，或者finalize方法已经被虚拟机调用过，虚拟机将这两种情况都视为“没有必要执行”

## 15、Session/Cookie的区别？ {#15、Session-Cookie的区别？}

Session存储在服务器端，类型可以是任意的Java对象，Cookie存储在客户端，类型只能为字符串

## 16、String/StringBuffer/StringBuilder的区别，扩展再问他们的实现？ {#16、String-StringBuffer-StringBuilder的区别，扩展再问他们的实现？}

String、StringBuffer是线程安全的，StringBuilder不是

String不继承任何类，StringBuffer、StringBuilder继承自AbstractStringBuilder

## 17、Servlet的生命周期？ {#17、Servlet的生命周期？}

Servlet生命周期分为三个阶段：

1、初始化阶段 调用init\(\)方法

2、响应客户请求阶段　　调用service\(\)方法

3、终止阶段　　调用destroy\(\)方法

Servlet初始化阶段：

在下列时刻Servlet容器装载Servlet：

1、Servlet容器启动时自动装载某些Servlet，实现它只需要在web.XML文件中的之间添加如下代码：

```
<loadon-startup>1</loadon-startup>
```

2、在Servlet容器启动后，客户首次向Servlet发送请求

3、Servlet类文件被更新后，重新装载Servlet

Servlet被装载后，Servlet容器创建一个Servlet实例并且调用Servlet的init\(\)方法进行初始化。在Servlet的整个生命周期内，init\(\)方法只被调用一次。

## 18、如何用Java分配一段连续的1G的内存空间？需要注意些什么？ {#18、如何用Java分配一段连续的1G的内存空间？需要注意些什么？}

```
ByteBuffer.allocateDirect(1024*1024*1024);
```

要注意内存溢出的问题

## 19、Java有自己的内存回收机制，但为什么还存在内存泄漏的问题呢？ {#19、Java有自己的内存回收机制，但为什么还存在内存泄漏的问题呢？}

首先内存泄漏 memory leak，是指程序在申请内存后，无法释放已申请的内存空间，一次内存泄漏危害可以忽略，但内存泄漏堆积后果很严重，无论多少内存,迟早会被占光。

比如下面这段代码，list持有o的引用，o暂时是无法被JVM垃圾回收的，只有当list被垃圾回收或者o从对象list删除掉后，o才能被JVM垃圾回收。

```
List<Object> list = new ArrayList<>();
Object o = new Object();
list.add(o);
o = null;
```

## 20、什么是java序列化，如何实现java序列化?\(写一个实例\)？ {#20、什么是java序列化，如何实现java序列化-写一个实例-？}

Java序列化就是把对象扁平化成一组数据，通过这组数据可以反序列化回Java对象。

例：序列化二叉树

```
private int index = -1;
String Serialize(TreeNode root) {
    StringBuilder sb = new StringBuilder();
    if (root == null) {
        sb.append("#,");
        return sb.toString();
    }
    sb.append(root.val + ",");
    sb.append(Serialize(root.left));
    sb.append(Serialize(root.right));
    return sb.toString();
}
TreeNode Deserialize(String str) {
    index++;
    String[] strs = str.split(",");
    if (index > strs.length || strs[index].equals("#")) {
        return null;
    }
    TreeNode root = new TreeNode(Integer.parseInt(strs[index]));
    root.left = Deserialize(str);
    root.right = Deserialize(str);
    return root;
}
```

## 21、String s = new String\(“abc”\);创建了几个 String Object? {#21、String-s-new-String-“abc”-创建了几个-String-Object}

2个，会创建String对象存放在字符串常量池跟堆中。

