## HTTP2.0

HTTP 2.0是在SPDY（An experimental protocol for a faster web, The Chromium Projects）基础上形成的下一代互联网通信协议。HTTP/2 的目的是通过支持请求与响应的多路复用来较少延迟，通过压缩HTTPS首部字段将协议开销降低，同时增加请求优先级和服务器端推送的支持。

本文目的是学习HTTP 2.0的原理并研究其通信的详细细节。大部分知识点源于《Web性能权威指南》。

1. 二进制分帧层

二进制分帧层，是HTTP 2.0性能增强的核心。

HTTP 1.x在应用层以纯文本的形式进行通信，而HTTP 2.0将所有的传输信息分割为更小的消息和帧，并对它们**采用二进制格式编码**。这样，客户端和服务端都需要引入新的二进制编码和解码的机制。

如下图所示，HTTP 2.0并没有改变HTTP 1.x的语义，只是在应用层使用二进制分帧方式传输。

![](/assets/20170405172818292.png)

因此，也引入了新的通信单位：

## **1.1 帧（frame）** {#11-帧frame}

HTTP 2.0通信的最小单位，包括帧首部、流标识符、优先值和帧净荷等。

![](/assets/20170405153816267.png)

其中，帧类型又可以分为：

DATA：用于传输HTTP消息体；

HEADERS：用于传输首部字段；

SETTINGS：用于约定客户端和服务端的配置数据。比如设置初识的双向流量控制窗口大小；

WINDOW\_UPDATE：用于调整个别流或个别连接的流量

PRIORITY： 用于指定或重新指定引用资源的优先级。

RST\_STREAM： 用于通知流的非正常终止。

PUSH\_ PROMISE： 服务端推送许可。

PING： 用于计算往返时间，执行“ 活性” 检活。

GOAWAY： 用于通知对端停止在当前连接中创建流。

标志位用于不同的帧类型定义特定的消息标志。比如DATA帧就可以使用End Stream: true表示该条消息通信完毕。流标识位表示帧所属的流ID。优先值用于HEADERS帧，表示请求优先级。R表示保留位。

下面是Wireshark抓包的一个DATA帧：

![](/assets/20170405170457247.png)

1.2 消息（message）

消息是指逻辑上的HTTP消息（请求/响应）。一系列数据帧组成了一个完整的消息。比如一系列DATA帧和一个HEADERS帧组成了请求消息。

1.3 流（stream）

流是连接中的一个虚拟信道，可以承载双向消息传输。每个流有唯一整数标识符。为了防止两端流ID冲突，客户端发起的流具有奇数ID，服务器端发起的流具有偶数ID。

所有HTTP 2. 0 通信都在一个TCP连接上完成， 这个连接可以承载任意数量的双向数据流Stream。 相应地， 每个数据流以 消息的形式发送， 而消息由一 或多个帧组成， 这些帧可以乱序发送， 然后根据每个帧首部的流标识符重新组装。

![](/assets/20170405172729697.png)

二进制分帧层保留了HTTP的语义不受影响，包括首部、方法等，在应用层来看，和HTTP 1.x没有差别。同时，所有同主机的通信能够在一个TCP连接上完成。

1. 多路复用共享连接

基于二进制分帧层，HTTP 2.0可以在共享TCP连接的基础上，同时发送请求和响应。HTTP消息被分解为独立的帧，而不破坏消息本身的语义，交错发送出去，最后在另一端根据流ID和首部将它们重新组合起来。

我们来对比下HTTP 1.x和HTTP 2.0，假设不考虑1.x的pipeline机制，双方四层都是一个TCP连接。客户端向服务度发起三个图片请求/image1.jpg，/image2.jpg，/image3.jpg。

HTTP 1.x发起请求是串行的，image1返回后才能再发起image2，image2返回后才能再发起image3。

![](/assets/20170406101003201.jpg)

HTTP 2.0建立一条TCP连接后，并行传输着3个数据流，客户端向服务端乱序发送stream1~3的一系列的DATA帧，与此同时，服务端已经在返回stream 1的DATA帧

![](/assets/20170406101019438.jpg)

性能对比，高下立见。HTTP 2.0成功解决了HTTP 1.x的队首阻塞问题（TCP层的阻塞仍无法解决），同时，也不需要通过pipeline机制多条TCP连接来实现并行请求与响应。减少了TCP连接数对服务器性能也有很大的提升。

1. 请求优先级

流可以带有一个31bit的优先级：

0：表示最高优先级

231-1：表示最低优先级

客户端明确指定优先级，服务端可以根据这个优先级作为依据交互数据，比如客户端优先级设置为.css&gt;.js&gt;.jpg（具体可参见《高性能网站建设指南》）， 服务端按优先级返回结果有利于高效利用底层连接，提高用户体验。

然而，也不能过分迷信请求优先级，仍然要注意以下问题：

服务端是否支持请求优先级

会否引起队首阻塞问题，比如高优先级的慢响应请求会阻塞其他资源的交互。

1. 服务端推送

HTTP 2.0增加了服务端推送功能，服务端可以根据客户端的请求，提前返回多个响应，推送额外的资源给客户端。如下图所示，客户端请求stream 1，/page.html。服务端在返回stream 1消息的同时推送了stream 2（/script.js）和stream 4（/style.css）。

![](/assets/20170406105233944.png)

PUSH\_PROMISE帧是服务端向客户端有意推送资源的信号。

如果客户端不需要服务端Push，可在SETTINGS帧中设定服务端流的值为0，禁用此功能

PUSH\_PROMISE帧中只包含预推送资源的首部。如果客户端对PUSH\_PROMISE帧没有意见，服务端在PUSH\_PROMISE帧后发送响应的DATA帧开始推送资源。如果客户端已经缓存该资源，不需要再推送，可以选择拒绝PUSH\_PROMISE帧。

PUSH\_PROMISE必须遵循请求-响应原则，只能借着对请求的响应推送资源。

目前，Apache的mod\_http2能够开启 H2Push on服务端推送Push。Nginx的ngx\_http\_v2\_module还不支持服务端Push。

```
 Apache mod_headers example
<Location /index.html>
    Header add Link "</css/site.css>;rel=preload"
    Header add Link "</images/logo.jpg>;rel=preload"
</Location>
```

# **5. 首部压缩** {#5-首部压缩}

HTTP 1.x每一次通信（请求/响应）都会携带首部信息用于描述资源属性。HTTP 2.0在客户端和服务端之间使用“首部表”来跟踪和存储之前发送的键-值对。首部表在连接过程中始终存在，新增的键-值对会更新到表尾，因此，不需要每次通信都需要再携带首部。

![](/assets/20170406151823031.png)

另外，HTTP 2.0使用了首部压缩技术，压缩算法使用HPACK。可让报头更紧凑，更快速传输，有利于移动网络环境。

需要注意的是，HTTP 2.0关注的是首部压缩，而我们常用的gzip等是报文内容（body）的压缩。二者不仅不冲突，且能够一起达到更好的压缩效果。

1. 一个完整的HTTP 2.0通信过程

考虑一个问题，客户端如何知道服务端是否支持HTTP 2.0？是否支持对二进制分帧层的编码和解码？所以，在两端使用HTTP 2.0通信之前，必然存在协议协商的过程。

6.1 基于ALPN的协商过程

支持HTTP 2.0的浏览器可以在TLS会话层自发完成和服务端的协议协商以确定是否使用HTTP 2.0通信。其原理是TLS 1.2中引入了扩展字段，以允许协议的扩展，其中ALPN协议（Application Layer Protocol Negotiation, 应用层协议协商, 前身是NPN）用于客户端和服务端的协议协商过程。

服务端使用ALPN，监听443端口默认提高HTTP 1.1，并允许协商其他协议，比如SPDY和HTTP 2.0。

比如，客户端在TLS握手Client Hello阶段表明自身支持HTTP 2.0

![](/assets/20170406154122927.png)

服务端收到后，响应Server Hello，表示自己也支持HTTP 2.0。双方开始HTTP 2.0通信。

![](/assets/20170406154450854.png)

6.2 基于HTTP的协商过程

然而，HTTP 2.0一定是HTTPS（TLS 1.2）的特权吗？

当然不是，客户端使用HTTP也可以开启HTTP 2.0通信。只不过因为HTTP 1. 0和HTTP 2. 0都使用同一个 端口（80）， 又没有服务器是否支持HTTP 2. 0的其他任何 信息，此时 客户端只能使用HTTP Upgrade机制（OkHttp, nghttp2等组件均可实现，也可以自己编码完成）通过协调确定适当的协议：

```
HTTP Upgrade request
GET / HTTP/1.1
host: nghttp2.org
connection: Upgrade, HTTP2-Settings
upgrade: h2c        /*发起带有HTTP2.0 Upgrade头部的请求*/       
http2-settings: AAMAAABkAAQAAP__   /*客户端SETTINGS净荷*/
user-agent: nghttp2/1.9.0-DEV

HTTP Upgrade response    
HTTP/1.1 101 Switching Protocols   /*服务端同意升级到HTTP 2.0*/
Connection: Upgrade
Upgrade: h2c

HTTP Upgrade success               /*协商完成*/
 ———————————————— 
版权声明：本文为CSDN博主「皖南笑笑生」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/zhuyiquan/article/details/69257126
```



