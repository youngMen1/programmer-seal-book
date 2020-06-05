# 1.基本介绍

#### 1.Serial收集器

#### 2.ParNew收集器

#### 3.Parallel Scavenge收集器

#### 4.Serial Old收集器

#### 5.Parallel Old收集器

#### 6.CMS收集器

#### 7.G1收集器

## 1.1.Serial收集器

这个收集器是一个单线程的收集器，在它进行垃圾收集时，必须暂停其他所有的工作线程，直到它线程结束。

### 特点

主要针对新生代；

```
  采用复制算法；

  单线程收集；mn

   进行垃圾收集时，必须暂停所有工作线程，直到完成；            

   即会"Stop The World"；

  Serial/Serial Old组合收集器运行示意图如下：
```

![](/static/image/20180611160921828.png)

## 1.2.ParNew收集器

## 1.3.Parallel Scavenge收集器

## 1.4.Serial Old收集器

## 1.5.Parallel Old收集器

## 1.6.CMS收集器

## 1.7.G1收集器



