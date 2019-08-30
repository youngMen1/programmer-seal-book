## select，poll，epoll优缺点及比较
### select优点
1）select()的可移植性更好，在某些Unix系统上不支持poll() 
2）select() 对于超时值提供了更好的精度：微秒，而poll是毫秒。
### select缺点 
1） 单个进程可监视的fd数量被限制。 
2） 需要维护一个用来存放大量fd的数据结构，这样会使得用户空间和内核空间在传递该结构时复制开销大。 
3） 对fd进行扫描时是线性扫描。fd剧增后，IO效率较低，因为每次调用都对fd进行线性扫描遍历，所以随着fd的增加会造成遍历速度慢的性能问题 
4）select() 函数的超时参数在返回时也是未定义的，考虑到可移植性，每次在超时之后在下一次进入到select之前都需要重新设置超时参数。
## poll
poll与select不同，通过一个pollfd数组向内核传递需要关注的事件，故没有描述符个数的限制，　 
　　pollfd中的events字段和revents分别用于标示关注的事件和发生的事件，故pollfd数组只需要被初始化一次。 
　　poll的实现机制与select类似，其对应内核中的sys_poll，只不过poll向内核传递pollfd数组，然后对pollfd中的每个描述符进行poll，相比处理fdset来说，poll效率更高。　 
　　poll返回后，需要对pollfd中的每个元素检查其revents值，来得指事件是否发生。