## 1、接口与抽象类的区别？

类可以实现多个接口但只能继承一个抽象类

接口里面所有的方法都是Public的，抽象类允许Private、Protected方法

JDK8前接口里面所有的方法都是抽象的且不允许有静态方法，抽象类可以有普通、静态方法，JDK8 接口可以实现默认方法和静态方法，前面加default、static关键字

## 2、Java中的异常有哪几类？分别怎么使用？ {#2、Java中的异常有哪几类？分别怎么使用？}

分为错误和异常，异常又包括运行时异常、非运行时异常

错误，如StackOverflowError、OutOfMemoryError

运行时异常，如NullPointerException、IndexOutOfBoundsException，都是RuntimeException及其子类

非运行时异常，如IOException、SQLException,都是Exception及其子类，这些异常是一定需要try catch捕获的



