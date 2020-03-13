## 1、使用两种命令创建一个文件？ {#1、使用两种命令创建一个文件？}

touch filename 建立一个空文件

cat &gt; filename 建立一文件，然后把接下来的键盘输入写入文件，直到按Ctrl+D为止

## 2、硬链接和软链接的区别？ {#2、硬链接和软链接的区别？}

硬链接：

1、文件有相同的 inode 及 data block；

2、只能对已存在的文件进行创建；

3、不能交叉文件系统进行硬链接的创建；

4、不能对目录进行创建，只可对文件创建；

5、删除一个硬链接文件并不影响其他有相同 inode 号的文件。

软连接：

1、软链接有自己的文件属性及权限等；

2、可对不存在的文件或目录创建软链接；

3、软链接可交叉文件系统；

4、软链接可对文件或目录创建；

5、创建软链接时，链接计数 i\_nlink 不会增加；

6、删除软链接并不影响被指向的文件，但若被指向的原文件被删除，则相关软连接被称为死链接（即 dangling link，若被指向路径文件被重新创建，死链接可恢复为正常的软链接）。

## 3、Linux常用命令有哪些？ {#3、Linux常用命令有哪些？}

[初窥Linux之我最常用的20条命令](http://blog.csdn.net/ljianhui/article/details/11100625/)

我的常用：

查找关闭端口进程`netstat -nlp | grep :3306kill pid`

删除文件`rm -rf`

查找日志`cat xx.log | grep 'xxx' | more`

解压tar.gz`tar -xzvf file.tar.gz`

创建文件`touch filename cat > filename`

修改文件`vi`

## 4、怎么看一个Java线程的资源耗用？ {#4、怎么看一个Java线程的资源耗用？}

linux下，所有的java内部线程，其实都对应了一个进程id，也就是说，linux上的jvm将java程序中的线程映射为操作系统进程。

1、`jps -lvm`或者`ps -ef | grep java`查看当前机器上运行的Java应用进程

2、`top -Hp pid`可以查看Java所有线程的资源耗用

4、`printf "%x\n" pid`等到线程ID的16进制

5、`jstack Java应用进程ID | grep 线程ID的16进制`

## 5、Load过高的可能性有哪些？ {#5、Load过高的可能性有哪些？}

[cpu load过高问题排查](https://www.cnblogs.com/lddbupt/p/5779655.html)

cpu load的飙升，一方面可能和full gc的次数增大有关，一方面可能和死循环有关系

## 6、/etc/hosts文件什么作用？ {#6、-etc-hosts文件什么作用？}

在当前主机给ip设置别名，通过该别名可以访问到该ip地址，通过别名、ip访问的效果是一样的

## 7、如何快速的将一个文本中所有“abc”替换为“xyz”？ {#7、如何快速的将一个文本中所有“abc”替换为“xyz”？}

`vi filename`编辑文本，按Esc键，输入 `:%s/abc/xyz/g`

## 8、如何在log文件中搜索找出error的日志？ {#8、如何在log文件中搜索找出error的日志？}

cat xx.log \| grep 'error'

## 9、发现磁盘空间不够，如何快速找出占用空间最大的文件？ {#9、发现磁盘空间不够，如何快速找出占用空间最大的文件？}

[Linux下查找大文件，大目录的方法](http://blog.csdn.net/Del_Zhu/article/details/52169442)

`find . -type f -size +100M | xargs du -h | sort -nr`

## 10、Java服务端问题排查（OOM，CPU高，Load高，类冲突） {#10、Java服务端问题排查（OOM，CPU高，Load高，类冲突）}

[java线上服务问题排查](http://blog.csdn.net/and1kaney/article/details/51214219)

## 11、Java常用问题排查工具及用法（top, iostat, vmstat, sar, tcpdump, jvisualvm, jmap, jconsole） {#11、Java常用问题排查工具及用法（top-iostat-vmstat-sar-tcpdump-jvisualvm-jmap-jconsole）}

[Java自带的性能监测工具用法简介——jstack、jconsole、jinfo、jmap、jdb、jsta、jvisualvm](http://blog.csdn.net/xad707348125/article/details/51985854)

## 12、Thread dump文件如何分析（Runnable，锁，代码栈，操作系统线程ID关联） {#12、Thread-dump文件如何分析（Runnable，锁，代码栈，操作系统线程ID关联）}

[性能分析之– JAVA Thread Dump 分析综述](http://blog.csdn.net/rachel_luo/article/details/8920596)

## 13、如何查看Java应用的线程信息？ {#13、如何查看Java应用的线程信息？}

参考

[4、怎么看一个Java线程的资源耗用？](https://blog.yk95.top/2018/01/30/成为Java顶尖程序员，先过了下面问题！（答案）/#4、怎么看一个Java线程的资源耗用？)

通过top命令拿到线程的pid后使用jstack命令

