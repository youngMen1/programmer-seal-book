在程序设计中，我们有时会遇到这样的情况，一个线程将数据写到一个buffer中，另外一个线程从中读数据。所以这里就有多线程竞争的问题。通常的解决办法是对竞争资源加锁。但是，一般加锁的损耗较高。其实，对于这样的一个线程写，一个线程读的特殊情况，可以以一种简单的无锁RingBuffer来实现。这样代码的运行效率很高。

本文借鉴了[Disruptor](http://code.google.com/p/disruptor/ )项目代码。

代码我在github上放了一份，需要的同学可以去下载（[RingBuffer.java](https://github.com/drneverend/buffers/blob/master/ringbuffer/RingBuffer.java)）。本文最后也会附上一份。

代码的基本原理如下。

![img](/static/image/221217446878287.png)

如图所示，假定buffer的长度是bufferSize. 我们设置两个指针。head指向的是下一次读的位置，而tail指向的是下一次写的位置。由于这里是环形buffer \(ring buffer\)，这里就有一个问题，怎样判断buffer是满或者空。这里采用的规则是，buffer的最后一个单元不存储数据。所以，如果head == tail，那么说明buffer为空。如果 head == tail + 1 \(mod bufferSize\)，那么说明buffer满了。

接下来就是最重要的内容了：怎样以无锁的方式进行线程安全的buffer的读写操作。基本原理是这样的。在进行读操作的时候，我们只修改head的值，而在写操作的时候我们只修改tail的值。在写操作时，我们在写入内容到buffer之后才修改tail的值；而在进行读操作的时候，我们会读取tail的值并将其赋值给copyTail。赋值操作是原子操作。所以在读到copyTail之后，从head到copyTail之间一定是有数据可以读的，不会出现数据没有写入就进行读操作的情况。同样的，读操作完成之后，才会修改head的数值；而在写操作之前会读取head的值判断是否有空间可以用来写数据。所以，这时候tail到head - 1之间一定是有空间可以写数据的，而不会出现一个位置的数据还没有读出就被写操作覆盖的情况。这样就保证了RingBuffer的线程安全性。

最后附上代码供参考。欢迎批评指正，也欢迎各种讨论！

```
public class RingBuffer {

    private final static int bufferSize = 1024;
    private String[] buffer = new String[bufferSize];
    private int head = 0;
    private int tail = 0;

    private Boolean empty() {
        return head == tail;
    }
    private Boolean full() {
        return (tail + 1) % bufferSize == head;
    }
    public Boolean put(String v) {
        if (full()) {
            return false;
        }
        buffer[tail] = v;
        tail = (tail + 1) % bufferSize;
        return true;
    }
    public String get() {
        if (empty()) {
            return null;
        }
        String result = buffer[head];
        head = (head + 1) % bufferSize;
        return result;
    }
    public String[] getAll() {
        if (empty()) {
            return new String[0];
        }
        int copyTail = tail;
        int cnt = head < copyTail ? copyTail - head : bufferSize - head + copyTail;
        String[] result = new String[cnt];
        if (head < copyTail) {
            for (int i = head; i < copyTail; i++) {
                result[i - head] = buffer[i];
            }
        } else {
            for (int i = head; i < bufferSize; i++) {
                result[i - head] = buffer[i];
            }
            for (int i = 0; i < copyTail; i++) {
                result[bufferSize - head + i] = buffer[i];
            }
        }
        head = copyTail;
        return result;
    }
}
```



