## 1.Java对象为啥要实现Serializable接口

## 1.1.问题

* 为什么定义Java实体对象时“implements Serializable”要实现Serializable接口?它的底层原理是什么?

* 为什么一定要序列化，序列化又是什么?

# 2.**Serializable接口概述**

实现了Serializable接口的类可以被ObjectOutputStream转换为字节流，同时也可以通过ObjectInputStream再将其解析为对象。

