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

