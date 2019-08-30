## select，poll，epoll优缺点及比较
### select优点
1）select()的可移植性更好，在某些Unix系统上不支持poll() 
2）select() 对于超时值提供了更好的精度：微秒，而poll是毫秒。
### select缺点 