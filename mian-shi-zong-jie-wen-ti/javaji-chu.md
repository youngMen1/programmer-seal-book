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

ArrayList：内部使用数组的形式实现了存储，利用数组的下标进行元素的访问，因此对元素的随机访问速度非常快。  
因为是数组，所以ArrayList在初始化的时候，有初始大小10，插入新元素的时候，会判断是否需要扩容，扩容的步长是0.5倍原容量，扩容方式是利用数组的复制，因此有一定的开销。

LinkedList：内部使用双向链表的结构实现存储，LinkedList有一个内部类作为存放元素的单元，里面有三个属性，用来存放元素本身以及前后2个单元的引用，另外LinkedList内部还有一个header属性，用来标识起始位置，LinkedList的第一个单元和最后一个单元都会指向header，因此形成了一个双向的链表结构。

ArrayList查找较快，插入、删除较慢，LinkedList查找较慢，插入、删除较快。

## 5、内存溢出是怎么回事？请举一个例子？ {#5、内存溢出是怎么回事？请举一个例子？}



