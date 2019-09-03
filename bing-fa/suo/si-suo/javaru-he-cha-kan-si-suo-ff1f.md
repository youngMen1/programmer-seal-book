Java如何查看死锁？

Java中当我们的开发涉及到多线程的时候，这个时候就很容易遇到死锁问题，刚开始遇到死锁问题的时候，我们很容易觉得莫名其妙，而且定位问题也很困难。

因为涉及到java多线程的时候，有的问题会特别复杂，而且就算我们知道问题出现是因为死锁了，我们也很难弄清楚为什么发生死锁，那么当我们遇到了死锁问题，我们应该如何来检测和查看死锁呢？

Java中jdk 给我们提供了很便利的工具，帮助我们定位和分析死锁问题：

1、死锁产生原因：当两个或者多个线程互相持有一定资源，并互相等待其他线程释放资源而形成的一种僵局，就是死锁。

2、构建一个死锁的场景：

```
public class Test {

    public static void main(String[] args) {
        new Thread(new Runnable() {

            @Override
            public void run() {
                synchronized (B.class) {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    synchronized (A.class) {

                    }
                }
            }
        }).start();
        new Thread(new Runnable() {

            @Override
            public void run() {
                synchronized (A.class) {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }
                    synchronized (B.class) {

                    }
                }

            }
        }).start();
    }

}
class A {

}

class B {

}
```

可以看到运行时，一个线程持有A资源，希望使用B资源，而另一个线程持有B资源，希望使用A 资源，然后就陷入了相互等待的僵局，这样就形成了死锁。

3、Jconsole查看死锁

进入java安装的位置，输入Jconsole，然后弹出界面（或者进入安装目录/java/jdk1.70\_80/bin/，点击Jconsole.exe）：

20160829115018823.png





