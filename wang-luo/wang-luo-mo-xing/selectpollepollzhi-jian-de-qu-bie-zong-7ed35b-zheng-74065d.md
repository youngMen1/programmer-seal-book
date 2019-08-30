# [select、poll、epoll之间的区别总结\[整理\]](https://www.cnblogs.com/Anker/p/3265058.html)
select，poll，epoll都是IO多路复用的机制。I/O多路复用就通过一种机制，可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作。但select，poll，epoll本质上都是同步I/O，因为他们都需要在读写事件就绪后自己负责进行读写，也就是说这个读写过程是阻塞的，而异步I/O则无需自己负责进行读写，异步I/O的实现会负责把数据从内核拷贝到用户空间。关于这三种IO多路复用的用法，前面三篇总结写的很清楚，并用服务器回射echo程序进行了测试。连接如下所示：

select：http://www.cnblogs.com/Anker/archive/2013/08/14/3258674.html

poll：http://www.cnblogs.com/Anker/archive/2013/08/15/3261006.html

epoll：http://www.cnblogs.com/Anker/archive/2013/08/17/3263780.html

今天对这三种IO多路复用进行对比，参考网上和书上面的资料，整理如下：


