## 死锁

死锁是两个或更多线程阻塞着等待其它处于死锁状态的线程所持有的锁。死锁通常发生在多个线程同时但以不同的顺序请求同一组锁的时候。

例如，如果线程1锁住了A，然后尝试对B进行加锁，同时线程2已经锁住了B，接着尝试对A进行加锁，这时死锁就发生了。线程1永远得不到B，线程2也永远得不到A，并且它们永远也不会知道发生了这样的事情。为了得到彼此的对象（A和B），它们将永远阻塞下去。这种情况就是一个死锁。

该情况如下：

```
Thread 1  locks A, waits for B
Thread 2  locks B, waits for A
```

#### 更复杂的死锁 {#ComplicatedDeadlock}

死锁可能不止包含2个线程，这让检测死锁变得更加困难。下面是4个线程发生死锁的例子：

```
Thread 1  locks A, waits for B
Thread 2  locks B, waits for C
Thread 3  locks C, waits for D
Thread 4  locks D, waits for A
```

线程1等待线程2，线程2等待线程3，线程3等待线程4，线程4等待线程1。

#### 数据库的死锁 {#DatabaseDeadlock}

更加复杂的死锁场景发生在数据库事务中。一个数据库事务可能由多条SQL更新请求组成。当在一个事务中更新一条记录，这条记录就会被锁住避免其他事务的更新请求，直到第一个事务结束。同一个事务中每一个更新请求都可能会锁住一些记录。当多个事务同时需要对一些相同的记录做更新操作时，就很有可能发生死锁，例如：

```
Transaction 1, request 1, locks record 1 for update
Transaction 2, request 1, locks record 2 for update
Transaction 1, request 2, tries to lock record 2 for update.
Transaction 2, request 2, tries to lock record 1 for update.
```

因为锁发生在不同的请求中，并且对于一个事务来说不可能提前知道所有它需要的锁，因此很难检测和避免数据库事务中的死锁。

死锁demo：

```
package thread;
/**
 * 
 * @author zhangliang
 *
 * 2016年4月12日 下午5:51:54
 */
public class DeadLocakTest {
    final static Object obj1 = new Object();
    final static Object obj2 = new Object();

    public static void main(String[] args) {
        // TODO Auto-generated method stub
        DeadLocakTest test = new DeadLocakTest();
        Thread t1 = new Thread(test.new DeadThread1());
        Thread t2 = new Thread(test.new DeadThread2());

        t1.start();
        t2.start();
    }


    class DeadThread1 implements Runnable {

        @Override
        public void run() {
            // TODO Auto-generated method stub
            synchronized(obj1){

                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName()+"loack obj1");
                synchronized(obj2){
                    System.out.println(Thread.currentThread().getName()+"is running");
                }

            }
        }  

     }

     class DeadThread2 implements Runnable {

            @Override
            public void run() {
                // TODO Auto-generated method stub
                synchronized(obj2){

                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName()+"loack obj2");
                    synchronized(obj1){
                        System.out.println(Thread.currentThread().getName()+"is running");
                    }

                }
            }  

         }

}
```

## 死锁检测：

我们上面的程序在eclipse运行会卡住，为了模拟线上情况，在Linux环境部署。

运行如下：

![](https://img-blog.csdn.net/20160412181614426?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

会卡住，我们解析来找到正在运行的程序id.使用jsatck来查看堆栈信息：

20160412181706647.png

具体见下面：

```
jstack -l 6081
2016-04-12 02:25:29
Full thread dump Java HotSpot(TM) 64-Bit Server VM (24.60-b09 mixed mode):

"Attach Listener" daemon prio=10 tid=0x00007fe8a4001000 nid=0x181e waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"DestroyJavaVM" prio=10 tid=0x00007fe8c0008800 nid=0x17c2 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"Thread-1" prio=10 tid=0x00007fe8c00a0800 nid=0x17cc waiting for monitor entry [0x00007fe8c5031000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at DeadLocakTest$DeadThread2.run(DeadLocakTest.java:61)
        - waiting to lock <0x00000000af24b248> (a java.lang.Object)
        - locked <0x00000000af24b258> (a java.lang.Object)
        at java.lang.Thread.run(Thread.java:745)

   Locked ownable synchronizers:
        - None

"Thread-0" prio=10 tid=0x00007fe8c009e800 nid=0x17cb waiting for monitor entry [0x00007fe8c5132000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at DeadLocakTest$DeadThread1.run(DeadLocakTest.java:38)
        - waiting to lock <0x00000000af24b258> (a java.lang.Object)
        - locked <0x00000000af24b248> (a java.lang.Object)
        at java.lang.Thread.run(Thread.java:745)

   Locked ownable synchronizers:
        - None

"Service Thread" daemon prio=10 tid=0x00007fe8c008a800 nid=0x17c9 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"C2 CompilerThread1" daemon prio=10 tid=0x00007fe8c0088000 nid=0x17c8 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"C2 CompilerThread0" daemon prio=10 tid=0x00007fe8c0085800 nid=0x17c7 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"Signal Dispatcher" daemon prio=10 tid=0x00007fe8c0084000 nid=0x17c6 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"Finalizer" daemon prio=10 tid=0x00007fe8c0064800 nid=0x17c5 in Object.wait() [0x00007fe8c5738000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x00000000af205630> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:135)
        - locked <0x00000000af205630> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:151)
        at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:209)

   Locked ownable synchronizers:
        - None

"Reference Handler" daemon prio=10 tid=0x00007fe8c0062800 nid=0x17c4 in Object.wait() [0x00007fe8c5839000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x00000000af2051b8> (a java.lang.ref.Reference$Lock)
        at java.lang.Object.wait(Object.java:503)
        at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:133)
        - locked <0x00000000af2051b8> (a java.lang.ref.Reference$Lock)

   Locked ownable synchronizers:
        - None

"VM Thread" prio=10 tid=0x00007fe8c005e000 nid=0x17c3 runnable 

"VM Periodic Task Thread" prio=10 tid=0x00007fe8c0095800 nid=0x17ca waiting on condition 

JNI global references: 106


Found one Java-level deadlock:
=============================
"Thread-1":
  waiting to lock monitor 0x00007fe8b0004e28 (object 0x00000000af24b248, a java.lang.Object),
  which is held by "Thread-0"
"Thread-0":
  waiting to lock monitor 0x00007fe8b00062c8 (object 0x00000000af24b258, a java.lang.Object),
  which is held by "Thread-1"

Java stack information for the threads listed above:
===================================================
"Thread-1":
        at DeadLocakTest$DeadThread2.run(DeadLocakTest.java:61)
        - waiting to lock <0x00000000af24b248> (a java.lang.Object)
        - locked <0x00000000af24b258> (a java.lang.Object)
        at java.lang.Thread.run(Thread.java:745)
"Thread-0":
        at DeadLocakTest$DeadThread1.run(DeadLocakTest.java:38)
        - waiting to lock <0x00000000af24b258> (a java.lang.Object)
        - locked <0x00000000af24b248> (a java.lang.Object)
        at java.lang.Thread.run(Thread.java:745)

Found 1 deadlock.
```

可以看到发生了死锁。

## 如何避免死锁：

在有些情况下死锁是可以避免的。

#### 加锁顺序 {#ordering}

当多个线程需要相同的一些锁，但是按照不同的顺序加锁，死锁就很容易发生。

如果能确保所有的线程都是按照相同的顺序获得锁，那么死锁就不会发生。看下面这个例子：

```
Thread 1:
  lock A 
  lock B

Thread 2:
   wait for A
   lock C (when A locked)

Thread 3:
   wait for A
   wait for B
   wait for C
```

如果一个线程（比如线程3）需要一些锁，那么它必须按照确定的顺序获取锁。它只有获得了从顺序上排在前面的锁之后，才能获取后面的锁。

例如，线程2和线程3只有在获取了锁A之后才能尝试获取锁C\(_译者注：获取锁A是获取锁C的必要条件_\)。因为线程1已经拥有了锁A，所以线程2和3需要一直等到锁A被释放。然后在它们尝试对B或C加锁之前，必须成功地对A加了锁。

按照顺序加锁是一种有效的死锁预防机制。但是，这种方式需要你事先知道所有可能会用到的锁\(_译者注：并对这些锁做适当的排序_\)，但总有些时候是无法预知的。

#### 加锁时限 {#timeout}

另外一个可以避免死锁的方法是在尝试获取锁的时候加一个超时时间，这也就意味着在尝试获取锁的过程中若超过了这个时限该线程则放弃对该锁请求。若一个线程没有在给定的时限内成功获得所有需要的锁，则会进行回退并释放所有已经获得的锁，然后等待一段随机的时间再重试。这段随机的等待时间让其它线程有机会尝试获取相同的这些锁，并且让该应用在没有获得锁的时候可以继续运行\(_译者注：加锁超时后可以先继续运行干点其它事情，再回头来重复之前加锁的逻辑_\)。

以下是一个例子，展示了两个线程以不同的顺序尝试获取相同的两个锁，在发生超时后回退并重试的场景：

```
Thread 1 locks A
```

```
Thread 2 locks B

Thread 1 attempts to lock B but is blocked
Thread 2 attempts to lock A but is blocked

Thread 1's lock attempt on B times out
Thread 1 backs up and releases A as well
Thread 1 waits randomly (e.g. 257 millis) before retrying.

Thread 2's lock attempt on A times out
Thread 2 backs up and releases B as well
Thread 2 waits randomly (e.g. 43 millis) before retrying.
```

在上面的例子中，线程2比线程1早200毫秒进行重试加锁，因此它可以先成功地获取到两个锁。这时，线程1尝试获取锁A并且处于等待状态。当线程2结束时，线程1也可以顺利的获得这两个锁（除非线程2或者其它线程在线程1成功获得两个锁之前又获得其中的一些锁）。

需要注意的是，由于存在锁的超时，所以我们不能认为这种场景就一定是出现了死锁。也可能是因为获得了锁的线程（导致其它线程超时）需要很长的时间去完成它的任务。

此外，如果有非常多的线程同一时间去竞争同一批资源，就算有超时和回退机制，还是可能会导致这些线程重复地尝试但却始终得不到锁。如果只有两个线程，并且重试的超时时间设定为0到500毫秒之间，这种现象可能不会发生，但是如果是10个或20个线程情况就不同了。因为这些线程等待相等的重试时间的概率就高的多（或者非常接近以至于会出现问题）。  
\(_译者注：超时和重试机制是为了避免在同一时间出现的竞争，但是当线程很多时，其中两个或多个线程的超时时间一样或者接近的可能性就会很大，因此就算出现竞争而导致超时后，由于超时时间一样，它们又会同时开始重试，导致新一轮的竞争，带来了新的问题。_\)

这种机制存在一个问题，在Java中不能对synchronized同步块设置超时时间。你需要创建一个自定义锁，或使用Java5中java.util.concurrent包下的工具。写一个自定义锁类不复杂，但超出了本文的内容。后续的Java并发系列会涵盖自定义锁的内容。

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

总结，出现死锁还是顺序容易出问题，也比较难调试。尽量保证顺序访问。这块还需要进一步结合锁来加深体验。

