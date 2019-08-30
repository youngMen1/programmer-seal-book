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

```
struct kevent {
    uintptr_t       ident;          /* identifier for this event，比如该事件关联的文件描述符 */
    int16_t         filter;         /* filter for event，可以指定监听类型，如EVFILT_READ，EVFILT_WRITE，EVFILT_TIMER等 */
    uint16_t        flags;          /* general flags ，可以指定事件操作类型，比如EV_ADD，EV_ENABLE， EV_DELETE等 */
    uint32_t        fflags;         /* filter-specific flags */
    intptr_t        data;           /* filter-specific data */
    void            *udata;         /* opaque user data identifier，可以携带的任意数据 */
};
```

**EV\_SET**

是用于初始化kevent结构的便利宏，其签名为:

```
EV_SET(&kev, ident, filter, flags, fflags, data, udata);
```

可以发现和kevent结构体完全对应，除了第一个，它就是你要初始化的那个kevent结构。

**kevent**是真正进行IO复用的函数，其签名为：

```
int kevent(int kq, 
    const struct kevent *changelist, // 监视列表
    int nchanges, // 长度
    struct kevent *eventlist, // kevent函数用于返回已经就绪的事件列表
    int nevents, // 长度
    const struct timespec *timeout); // 超时限制
```

# 4. 完整示例 {#完整示例}

下面给出一个完整的示例，这个程序将从标准输入中读取数据，写到标准输出中。其中输入输出全部使用kqueue来进行IO复用。可以使用重定向把文件写入标准输入来进行测试。

```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/event.h>
#include <errno.h>
#include <string.h>

// 为文件描述符打开对应状态位的工具函数
void turn_on_flags(int fd, int flags){
    int current_flags;
    // 获取给定文件描述符现有的flag
    // 其中fcntl的第二个参数F_GETFL表示要获取fd的状态
    if( (current_flags = fcntl(fd, F_GETFL)) < 0 ) exit(1);

    // 施加新的状态位
    current_flags |= flags;
    if( fcntl(fd, F_SETFL, current_flags) < 0 ) exit(1);
}

// 错误退出的工具函数
int quit(const char *msg){
    perror(msg);
    exit(1);
}

const static int FD_NUM = 2; // 两个文件描述符，分别为标准输入与输出
const static int BUFFER_SIZE = 1024; // 缓冲区大小

// 完全以IO复用的方式读入标准输入流数据，输出到标准输出流中
int main(){
    struct kevent changes[FD_NUM];
    struct kevent events[FD_NUM];

    // 创建一个kqueue
    int kq;
    if( (kq = kqueue()) == -1 ) quit("kqueue()");

    // 准备从标准输入流中读数据
    int stdin_fd = STDIN_FILENO;
    int stdout_fd = STDOUT_FILENO;

    // 设置为非阻塞
    turn_on_flags(stdin_fd, O_NONBLOCK);
    turn_on_flags(stdout_fd, O_NONBLOCK);

    // 注册监听事件
    int k = 0;
    EV_SET(&changes[k++], stdin_fd, EVFILT_READ, EV_ADD | EV_ENABLE, 0, 0, &stdin_fd);
    EV_SET(&changes[k++], stdout_fd, EVFILT_WRITE, EV_ADD | EV_ENABLE, 0, 0, &stdout_fd);

    int nev, nread, nwrote = 0; // 发生事件的数量, 已读字节数, 已写字节数
    char buffer[BUFFER_SIZE];

    while(1){
        nev = kevent(kq, changes, FD_NUM, events, FD_NUM, NULL); // 已经就绪的文件描述符数量
        if( nev <= 0 ) quit("kevent()");

        int i;
        for(i=0; i<nev; i++){
            struct kevent event = events[i];
            if( event.flags & EV_ERROR ) quit("Event error");

            int ev_fd = *((int *)event.udata);

            // 输入流就绪 且 缓冲区还有空间能继续读
            if( ev_fd == stdin_fd && nread < BUFFER_SIZE ){
                int new_nread;
                if( (new_nread = read(ev_fd, buffer + nread, sizeof(buffer) - nread)) <= 0 )
                    quit("read()"); // 由于可读事件已经发生，因此如果读出0个字节也是不正常的

                nread += new_nread; // 递增已读数据字节数
            }

            // 输出流就绪 且 缓冲区有内容可以写出
            if( ev_fd == stdout_fd && nread > 0 ){
                if( (nwrote = write(stdout_fd, buffer, nread)) <=0 )
                    quit("write()");

                memmove(buffer, buffer+nwrote, nwrote); // 为了使实现的代码更简洁，这里把还没有写出去的数据往前移动
                nread -= nwrote; // 减去已经写出去的字节数
            }
        }
    }

    return 0;
}
```

程序中对stdin和stdout设置非阻塞的原因是我们希望有多少就绪的数据就读多少，或者能写入多少进缓冲区就写入多少。否则在阻塞模式下，如果read没有填满buffer（文件没读完时）,或者还有buffer数据没写入时，系统调用\(read和write\)会阻塞，这会对性能造成很大影响。因此这里设置为非阻塞模式。

