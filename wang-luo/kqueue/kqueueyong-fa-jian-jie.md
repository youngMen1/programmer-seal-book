## [kqueue用法简介](#)

# 1.什么是kqueue和IO复用 {#什么是kqueue和io复用}

kueue是在UNIX上比较高效的IO复用技术。  
所谓的IO复用，就是同时等待多个文件描述符就绪，以系统调用的形式提供。如果所有文件描述符都没有就绪的话，该系统调用阻塞，否则调用返回，允许用户进行后续的操作。  
常见的IO复用技术有select, poll, epoll以及kqueue等等。其中epoll为Linux独占，而kqueue则在许多UNIX系统上存在，包括OS X（好吧，现在叫macOS了。。）

# 2. 使用概览 {#使用概览}

kueue在设计上是非常简洁的，在易用性上可能比select和epoll更好一些。  
使用kqueue的大致代码如下：（后面会给出一个完整的示例）

```
const static int FD_NUM = 2 // 要监视多少个文件描述符

int kq = kqueue(); // kqueue对象

// kqueue的事件结构体，不需要直接操作
struct kevent changes[FD_NUM]; // 要监视的事件列表
struct kevent events[FD_NUM]; // kevent返回的事件列表（参考后面的kevent函数）

int stdin_fd = STDIN_FILENO;
int stdout_fd = STDOUT_FILENO;

// 在changes列表中注册标准输入流的读事件 以及 标准输出流的写事件
// 最后一个参数可以是任意的附加数据（void * 类型），在这里给事件附上了当前的文件描述符，后面会用到
EV_SET(&changes[0], stdin_fd, EVFILT_READ, EV_ADD | EV_ENABLE, 0, 0, &stdin_fd); 
EV_SET(&changes[1], stdout_fd, EVFILT_WRITE, EV_ADD | EV_ENABLE, 0, 0, &stdin_fd);

// 进行kevent函数调用，如果changes列表里有任何就绪的fd，则把该事件对应的结构体放进events列表里面
// 返回值是这次调用得到了几个就绪的事件 (nev = number of events)
int nev = kevent(kq, changes, FD_NUM, events, FD_NUM, NULL); // 已经就绪的文件描述符数量
for(int i=0; i<nev; i++){
    struct kevent event = events[i]; // 一个个取出已经就绪的事件

    int ready_fd = *((int *)event.udata); // 从附加数据里面取回文件描述符的值
    if( ready_fd == stdin_fd ){
        // 读取ready_fd
    }else if( ready_fd == stdin_fd ){
        // 写入ready_fd
    }
}
```

# 3. 相关结构体与函数解析 {#相关结构体与函数解析}

可以看出来，kqueue体系只有三样东西：struct kevent结构体，EV\_SET宏以及kevent函数。

**struct kevent**结构体内容如下：

