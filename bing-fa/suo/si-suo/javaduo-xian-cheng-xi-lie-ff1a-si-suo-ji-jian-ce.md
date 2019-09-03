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



