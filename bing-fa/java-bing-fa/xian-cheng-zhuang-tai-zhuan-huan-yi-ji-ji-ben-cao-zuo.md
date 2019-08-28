## 新建线程

一个java程序从main\(\)方法开始执行，然后按照既定的代码逻辑执行，看似没有其他线程参与，但实际上java程序天生就是一个多线程程序，包含了：

（1）分发处理发送给给JVM信号的线程；

（2）调用对象的finalize方法的线程；

（3）清除Reference的线程；

（4）main线程，用户程序的入口。那么，如何在用户程序中新建一个线程了，只要有三种方式：

1. 通过继承Thread类，重写run方法；
2. 通过实现runable接口；
3. 通过实现callable接口这三种方式，下面看具体demo。

```
public class CreateThreadDemo {

            public static void main(String[] args) {
                //1.继承Thread
                Thread thread = new Thread() {
                    @Override
                    public void run() {
                        System.out.println("继承Thread");
                        super.run();
                    }
                };
                thread.start();
                //2.实现runable接口
                Thread thread1 = new Thread(new Runnable() {
                    @Override
                    public void run() {
                        System.out.println("实现runable接口");
                    }
                });
                thread1.start();
                //3.实现callable接口
                ExecutorService service = Executors.newSingleThreadExecutor();
                Future<String> future = service.submit(new Callable() {
                    @Override
                    public String call() throws Exception {
                        return "通过实现Callable接口";
                    }
                });
                try {
                    String result = future.get();
                    System.out.println(result);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (ExecutionException e) {
                    e.printStackTrace();
                }
            }
        }
```

三种新建线程的方式具体看以上注释，需要主要的是：

* 由于java不能多继承可以实现多个接口，因此，在创建线程的时候尽量多考虑采用实现接口的形式；
* 实现callable接口，提交给ExecutorService返回的是异步执行的结果，另外，通常也可以利用FutureTask\(Callable&lt;V&gt; callable\)将callable进行包装然后FeatureTask提交给ExecutorsService。如图，

![](/assets/futureTask接口实现关系.png)

另外由于FeatureTask也实现了Runable接口也可以利用上面第二种方式（实现Runable接口）来新建线程；

* 可以通过Executors将Runable转换成Callable，具体方法是：Callable&lt;T&gt; callable\(Runnable task, T result\)， Callable&lt;Object&gt; callable\(Runnable task\)。

## 2. 线程状态转换

此图来源于《JAVA并发编程的艺术》一书中，线程是会在不同的状态间进行转换的，java线程线程转换图如上图所示。线程创建之后调用start\(\)方法开始运行，当调用wait\(\),join\(\),LockSupport.lock\(\)方法线程会进入到\*\*WAITING\*\*状态，而同样的

![](/assets/线程状态转换关系.png)

wait\(long timeout\)，

sleep\(long\),

join\(long\),

LockSupport.parkNanos\(\),

LockSupport.parkUtil\(\)增加了超时等待的功能，也就是调用这些方法后线程会进入\*\*TIMED\_WAITING\*\*状态，当超时等待时间到达后，线程会切换到Runable的状态，另外当WAITING和TIMED \_WAITING状态时可以通过Object.notify\(\),Object.notifyAll\(\)方法使线程转换到Runable状态。当线程出现资源竞争时，即等待获取锁的时候，线程会进入到\*\*BLOCKED\*\*阻塞状态，当线程获取锁时，线程进入到Runable状态。线程运行结束后，线程进入到\*\*TERMINATED\*\*状态，状态转换可以说是线程的生命周期。另外需要注意的是：

* 当线程进入到synchronized方法或者synchronized代码块时，线程切换到的是BLOCKED状态，而使用java.util.concurrent.locks下lock进行加锁的时候线程切换的是WAITING或者TIMED\_WAITING状态，因为lock会调用LockSupport的方法。

用一个表格将上面六种状态进行一个总结归纳。![](/assets/线程状态.png)

## 3. 线程状态的基本操作

除了新建一个线程外，线程在生命周期内还有需要基本操作，而这些操作会成为线程间一种通信方式，比如使用中断（interrupted）方式通知实现线程间的交互等等，下面就将具体说说这些操作。

### 3.1. interrupted

中断可以理解为线程的一个标志位，它表示了一个运行中的线程是否被其他线程进行了中断操作。中断好比其他线程对该线程打了一个招呼。其他线程可以调用该线程的interrupt\(\)方法对其进行中断操作，同时该线程可以调用

isInterrupted（）来感知其他线程对其自身的中断操作，从而做出响应。另外，同样可以调用Thread的静态方法

interrupted（）对当前线程进行中断操作，该方法会清除中断标志位。\*\*需要注意的是，当抛出InterruptedException时候，会清除中断标志位，也就是说在调用isInterrupted会返回false。\*\*

![](/assets/中断线程方法.png)

下面结合具体的实例来看一看

```
public class InterruptDemo {
        public static void main(String[] args) throws InterruptedException {
            //sleepThread睡眠1000ms
            final Thread sleepThread = new Thread() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    super.run();
                }
            };
            //busyThread一直执行死循环
            Thread busyThread = new Thread() {
                @Override
                public void run() {
                    while (true) ;
                }
            };
            sleepThread.start();
            busyThread.start();
            sleepThread.interrupt();
            busyThread.interrupt();
            while (sleepThread.isInterrupted()) ;
            System.out.println("sleepThread isInterrupted: " + sleepThread.isInterrupted());
            System.out.println("busyThread isInterrupted: " + busyThread.isInterrupted());
        }
    }
```

输出结果

&gt; sleepThread isInterrupted: false

&gt; busyThread isInterrupted: true

开启了两个线程分别为sleepThread和BusyThread, sleepThread睡眠1s，BusyThread执行死循环。然后分别对着两个线程进行中断操作，可以看出sleepThread抛出InterruptedException后清除标志位，而busyThread就不会清除标志位。

另外，同样可以通过中断的方式实现线程间的简单交互， while \(sleepThread.isInterrupted\(\)\) 表示在Main中会持续监测sleepThread，一旦sleepThread的中断标志位清零，即sleepThread.isInterrupted\(\)返回为false时才会继续Main线程才会继续往下执行。因此，中断操作可以看做线程间一种简便的交互方式。一般在\*\*结束线程时通过中断标志位或者标志位的方式可以有机会去清理资源，相对于武断而直接的结束线程，这种方式要优雅和安全。\*\*

## 3.2.    join

join方法可以看做是线程间协作的一种方式，很多时候，一个线程的输入可能非常依赖于另一个线程的输出，这就像两个好基友，一个基友先走在前面突然看见另一个基友落在后面了，这个时候他就会在原处等一等这个基友，等基友赶上来后，就两人携手并进。其实线程间的这种协作方式也符合现实生活。在软件开发的过程中，从客户那里获取需求后，需要经过需求分析师进行需求分解后，这个时候产品，开发才会继续跟进。如果一个线程实例A执行了threadB.join\(\),其含义是：当前线程A会等待threadB线程终止后threadA才会继续执行。关于join方法一共提供如下这些方法:

&gt; public final synchronized void join\(long millis\)

&gt; public final synchronized void join\(long millis, int nanos\)

&gt; public final void join\(\) throws InterruptedException

Thread类除了提供join\(\)方法外，另外还提供了超时等待的方法，如果线程threadB在等待的时间内还没有结束的话，threadA会在超时之后继续执行。join方法源码关键是：

```
while (isAlive()) {
        wait(0);
     }
```

可以看出来当前等待对象threadA会一直阻塞，直到被等待对象threadB结束后即isAlive\(\)返回false的时候才会结束while循环，当threadB退出时会调用notifyAll\(\)方法通知所有的等待线程。下面用一个具体的例子来说说join方法的使用：

```
public class JoinDemo {
	    public static void main(String[] args) {
	        Thread previousThread = Thread.currentThread();
	        for (int i = 1; i <= 10; i++) {
	            Thread curThread = new JoinThread(previousThread);
	            curThread.start();
	            previousThread = curThread;
	        }
	    }
	
	    static class JoinThread extends Thread {
	        private Thread thread;
	
	        public JoinThread(Thread thread) {
	            this.thread = thread;
	        }
	
	        @Override
	        public void run() {
	            try {
	                thread.join();
	                System.out.println(thread.getName() + " terminated.");
	            } catch (InterruptedException e) {
	                e.printStackTrace();
	            }
	        }
	    }
	}

输出结果为：

> main terminated.
> Thread-0 terminated.
> Thread-1 terminated.
> Thread-2 terminated.
> Thread-3 terminated.
> Thread-4 terminated.
> Thread-5 terminated.
> Thread-6 terminated.
> Thread-7 terminated.
> Thread-8 terminated.
```







