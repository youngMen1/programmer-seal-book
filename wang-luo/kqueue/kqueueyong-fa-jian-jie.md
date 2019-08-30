## [kqueue用法简介](#)

# 1.什么是kqueue和IO复用 {#什么是kqueue和io复用}

kueue是在UNIX上比较高效的IO复用技术。  
所谓的IO复用，就是同时等待多个文件描述符就绪，以系统调用的形式提供。如果所有文件描述符都没有就绪的话，该系统调用阻塞，否则调用返回，允许用户进行后续的操作。  
常见的IO复用技术有select, poll, epoll以及kqueue等等。其中epoll为Linux独占，而kqueue则在许多UNIX系统上存在，包括OS X（好吧，现在叫macOS了。。）

# 2. 使用概览 {#使用概览}

kueue在设计上是非常简洁的，在易用性上可能比select和epoll更好一些。  
使用kqueue的大致代码如下：（后面会给出一个完整的示例）



