## HTTP协议详解

**一、概念**

协议是指计算机通信网络中两台计算机之间进行通信所必须共同遵守的规定或规则，超文本传输协议\(HTTP\)是一种通信协议，它允许将超文本标记语言\(HTML\)文档从Web服务器传送到客户端的浏览器。

HTTP协议，即超文本传输协议\(Hypertext transfer protocol\)。是一种详细规定了浏览器和万维网\(WWW = World Wide Web\)服务器之间互相通信的规则，通过因特网传送万维网文档的数据传送协议。

HTTP协议是用于从WWW服务器传输超文本到本地浏览器的传送协议。它可以使浏览器更加高效，使网络传输减少。它不仅保证计算机正确快速地传输超文本文档，还确定传输文档中的哪一部分，以及哪部分内容首先显示\(如文本先于图形\)等。

HTTP是一个应用层协议，由请求和响应构成，是一个标准的客户端服务器模型。HTTP是一个无状态的协议。

在Internet中所有的传输都是通过TCP/IP进行的。HTTP协议作为TCP/IP模型中应用层的协议也不例外。HTTP协议通常承载于TCP协议之上，有时也承载于TLS或SSL协议层之上，这个时候，就成了我们常说的HTTPS。如下图所示：

![](/assets/https的图片.png)

HTTP默认的端口号为80，HTTPS的端口号为443。

浏览网页是HTTP的主要应用，但是这并不代表HTTP就只能应用于网页的浏览。HTTP是一种协议，只要通信的双方都遵守这个协议，HTTP就能有用武之地。比如咱们常用的QQ，迅雷这些软件，都会使用HTTP协议\(还包括其他的协议\)。

**二、简史**

它的发展是万维网协会（World Wide Web Consortium）和Internet工作小组IETF（Internet Engineering Task Force）合作的结果，（他们）最终发布了一系列的RFC，RFC 1945定义了HTTP/1.0版本。其中最著名的就是RFC 2616。RFC 2616定义了今天普遍使用的一个版本——HTTP 1.1。

**三、特点**

HTTP协议永远都是客户端发起请求，服务器回送响应。这样就限制了使用HTTP协议，无法实现在客户端没有发起请求的时候，服务器将消息推送给客户端。

HTTP协议的主要**特点**可概括如下：  
1、支持客户/服务器模式。支持基本认证和安全认证。  
2、简单快速：客户向服务器请求服务时，只需传送请求方法和路径。请求方法常用的有GET、HEAD、POST。每种方法规定了客户与服务器联系的类型不同。由于HTTP协议简单，使得HTTP服务器的程序规模小，因而通信速度很快。  
3、灵活：HTTP允许传输任意类型的数据对象。正在传输的类型由Content-Type加以标记。  
4、HTTP 0.9和1.0使用非持续连接：限制每次连接只处理一个请求，服务器处理完客户的请求，并收到客户的应答后，即断开连接。HTTP 1.1使用持续连接：不必为每个web对象创建一个新的连接，一个连接可以传送多个对象，采用这种方式可以节省传输时间。  
5、无状态：HTTP协议是无状态协议。无状态是指协议对于事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大。

**无状态协议：**  
协议的状态是指下一次传输可以“记住”这次传输信息的能力。  
http是不会为了下一次连接而维护这次连接所传输的信息,为了保证服务器内存。  
比如客户获得一张网页之后关闭浏览器，然后再一次启动浏览器，再登陆该网站，但是服务器并不知道客户关闭了一次浏览器。  
由于Web服务器要面对很多浏览器的并发访问，为了提高Web服务器对并发访问的处理能力，在设计HTTP协议时规定Web服务器发送HTTP应答报文和文档时，不保存发出请求的Web浏览器进程的任何状态信息。这有可能出现一个浏览器在短短几秒之内两次访问同一对象时，服务器进程不会因为已经给它发过应答报文而不接受第二期服务请求。由于Web服务器不保存发送请求的Web浏览器进程的任何信息，因此HTTP协议属于无状态协议（Stateless Protocol）。

**HTTP协议是无状态的和Connection: keep-alive的区别：**  
无状态是指协议对于事务处理没有记忆能力，服务器不知道客户端是什么状态。从另一方面讲，打开一个服务器上的网页和你之前打开这个服务器上的网页之间没有任何联系。  
HTTP是一个无状态的面向连接的协议，无状态不代表HTTP不能保持TCP连接，更不能代表HTTP使用的是UDP协议（无连接）。  
从HTTP/1.1起，默认都开启了Keep-Alive，保持连接特性，简单地说，当一个网页打开完成后，客户端和服务器之间用于传输HTTP数据的TCP连接不会关闭，如果客户端再次访问这个服务器上的网页，会继续使用这一条已经建立的连接。  
Keep-Alive不会永久保持连接，它有一个保持时间，可以在不同的服务器软件（如Apache）中设定这个时间。

**四、工作流程**

一次HTTP操作称为一个事务，其工作过程可分为四步：  
1）首先客户机与服务器需要建立连接。只要单击某个超级链接，HTTP的工作开始。  
2）建立连接后，客户机发送一个请求给服务器，请求方式的格式为：统一资源标识符（URL）、协议版本号，后边是MIME信息包括请求修饰符、客户机信息和可能的内容。  
3）服务器接到请求后，给予相应的响应信息，其格式为一个状态行，包括信息的协议版本号、一个成功或错误的代码，后边是MIME信息包括服务器信息、实体信息和可能的内容。  
4）客户端接收服务器所返回的信息通过浏览器显示在用户的显示屏上，然后客户机与服务器断开连接。  
如果在以上过程中的某一步出现错误，那么产生错误的信息将返回到客户端，有显示屏输出。对于用户来说，这些过程是由HTTP自己完成的，用户只要用鼠标点击，等待信息显示就可以了。

![](/assets/121703453173884.png)

HTTP是基于传输层的TCP协议，而TCP是一个端到端的面向连接的协议。所谓的端到端可以理解为进程到进程之间的通信。所以HTTP在开始传输之前，首先需要建立TCP连接，而TCP连接的过程需要所谓的“三次握手”。下图所示TCP连接的三次握手。

在TCP三次握手之后，建立了TCP连接，此时HTTP就可以进行传输了。一个重要的概念是面向连接，既HTTP在传输完成之间并不断开TCP连接。在HTTP1.1中\(通过Connection头设置\)这是默认行为。

![](/assets/122106327705778.png)

**五、使用Wireshark抓TCP、http包**

打开Wireshark，选择工具栏上的"Capture"-&gt;"Options"

![](/assets/121453326605793.png)![](/assets/121455326769362.png)

点击"Capture Filter"，此处选择的是"HTTP TCP port（80）"，选择后点击上图的"Start"开始抓包。

然后在浏览器中打开[http://image.baidu.com/，抓包结果如下图所示：](http://image.baidu.com/，抓包结果如下图所示：)

![](/assets/121502073952141.png)

在上图中，可清晰的看到客户端浏览器（ip为192.168.1.6）与服务器（115.239.210.36）的交互过程：

1）No1：浏览器（192.168.1.6）向服务器（115.239.210.36）发出连接请求。此为TCP三次握手第一步，此时从图中可以看出，为SYN，seq:X （x=0）；

2）No2：服务器（115.239.210.36）回应了浏览器（192.168.1.6）的请求，并要求确认，此时为：SYN，ACK，此时seq：y（y为0），ACK：x+1（为1）。此为三次握手的第二步；

3）No3：浏览器（192.168.1.6）回应了服务器（115.239.210.36）的确认，连接成功。为：ACK，此时seq：x+1（为1），ACK：y+1（为1）。此为三次握手的第三步；

4）No4：浏览器（192.168.1.6）发出一个页面HTTP请求；

5）No5：服务器（115.239.210.36）确认；

6）No6：服务器（115.239.210.36）发送数据；

7）No8：客户端浏览器（192.168.1.6）确认；

![](/assets/121521541766071.png)

8）No81：客户端（192.168.1.6）发出一个图片HTTP请求；  
9）No202：服务器（115.239.210.36）发送状态响应码200 OK。

**六、头域**

每个头域由一个域名，冒号（:）和域值三部分组成。域名是大小写无关的，域值前可以添加任何数量的空格符，头域可以被扩展为多行，在每行开始处，使用至少一个空格或制表符。

_**6.1、请求信息：**_  
发出的请求信息格式如下：  
●请求行，例如GET /images/logo.gif HTTP/1.1，表示从/images目录下请求logo.gif这个文件。  
●（请求）头，例如Accept-Language: en  
●空行  
●可选的消息体　请求行和标题必须以&lt;CR&gt;&lt;LF&gt;作为结尾（也就是，回车然后换行）。空行内必须只有&lt;CR&gt;&lt;LF&gt;而无其他空格。在HTTP/1.1协议中，所有的请求头，除post外，都是可选的。

![](/assets/121712545823346.png)

三个部分分别是：请求行、消息报头、请求正文。

_**6.2、请求方法**_  
HTTP/1.1协议中共定义了八种方法（有时也叫“动作”）来表明Request-URI指定的资源的不同操作方式：  
OPTIONS- 返回服务器针对特定资源所支持的HTTP请求方法。也可以利用向Web服务器发送'\*'的请求来测试服务器的功能性。  
HEAD- 向服务器索要与GET请求相一致的响应，只不过响应体将不会被返回。这一方法可以在不必传输整个响应内容的情况下，就可以获取包含在响应消息头中的元信息。该方法常用于测试超链接的有效性，是否可以访问，以及最近是否更新。  
GET- 向特定的资源发出请求。注意：GET方法不应当被用于产生“副作用”的操作中，例如在web app.中。其中一个原因是GET可能会被网络蜘蛛等随意访问。  
POST- 向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST请求可能会导致新的资源的建立和/或已有资源的修改。  
PUT- 向指定资源位置上传其最新内容。  
DELETE- 请求服务器删除Request-URI所标识的资源。  
TRACE- 回显服务器收到的请求，主要用于测试或诊断。  
CONNECT- HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器。  
PATCH- 用来将局部修改应用于某一资源，添加于规范RFC5789。  
方法名称是区分大小写的。当某个请求所针对的资源不支持对应的请求方法的时候，服务器应当返回状态码405（Method Not Allowed）；当服务器不认识或者不支持对应的请求方法的时候，应当返回状态码501（Not Implemented）。  
HTTP服务器至少应该实现GET和HEAD方法，其他方法都是可选的。此外，除了上述方法，特定的HTTP服务器还能够扩展自定义的方法。

**GET和POST的区别**：  
1、GET提交的数据会放在URL之后，以?分割URL和传输数据，参数之间以&相连，如EditPosts.aspx?name=test1&id=123456. POST方法是把提交的数据放在HTTP包的Body中。  
2、GET提交的数据大小有限制，最多只能有1024字节（因为浏览器对URL的长度有限制），而POST方法提交的数据没有限制。  
3、GET方式需要使用Request.QueryString来取得变量的值，而POST方式通过Request.Form来获取变量的值。  
4、GET方式提交数据，会带来安全问题，比如一个登录页面，通过GET方式提交数据时，用户名和密码将出现在URL上，如果页面可以被缓存或者其他人可以访问这台机器，就可以从历史记录获得该用户的账号和密码。

_**6.3、响应消息**_  
客户端向服务器发送一个请求，服务器以一个状态行作为响应，响应的内容包括：消息协议的版本、成功或者错误编码、服务器信息、实体元信息以及必要的实体内容。根据响应类别的类别，服务器响应里可以含实体内容，但不是所有的响应都有实体内容。  
响应头第一行也称为状态行，格式如下（下图中红线标出的那行）：  
HTTP-Version 空格 Status-Code 空格 Reason-Phrase CRLF  
HTTP- Version表示HTTP版本，例如为HTTP/1.1。Status- Code是结果代码，用三个数字表示。Reason-Phrase是个简单的文本描述，解释Status-Code的具体原因。Status-Code用于机器自动识别，Reason-Phrase用于人工理解。Status-Code的第一个数字代表响应类别，可能取5个不同的值。后两个数字没有分类作用。Status-Code的第一个数字代表响应的类别，后续两位描述在该类响应下发生的具体状况，具体请参见：HTTP状态码 。

![](/assets/121724042548791.png)

三个部分分别是：状态行、消息报头、响应正文。

无论你何时浏览一个网页，你的电脑都会通过一个使用HTTP协议的服务器来获取所请求的数据。在你请求的网页显示在浏览器之前，支配网页的网站服务器会返回一个包含有状态码的HTTP头文件。这个状态码提供了有关所请求网页的相关条件信息。如果一切正常，一个标准网页会收到一条诸如200的状态码。当然我们的目的不是去研究200响应码，而是去探讨那些代表出现错误信息的服务器头文件响应码，例如表示“未找到指定网页”的404码。

_**6.4、响应头域**_  
服务器需要传递许多附加信息，这些信息不能全放在状态行里。因此，需要另行定义响应头域，用来描述这些附加信息。响应头域主要描述服务器的信息和Request-URI的信息。

![](/assets/121538263329761.png)

_**6.5、HTTP常见的请求头**_（在HTTP/1.1 协议中，所有的请求头，除Host外，都是可选的）

If-Modified-Since：把浏览器端缓存页面的最后修改时间发送到服务器去，服务器会把这个时间与服务器上实际文件的最后修改时间进行对比。如果时间一致，那么返回304，客户端就直接使用本地缓存文件。如果时间不一致，就会返回200和新的文件内容。客户端接到之后，会丢弃旧文件，把新文件缓存起来，并显示在浏览器中。

![](/assets/121756006148567.png)

If-None-Match

：If-None-Match和ETag一起工作，工作原理是在HTTP Response中添加ETag信息。 当用户再次请求该资源时，将在HTTP Request 中加入If-None-Match信息\(ETag的值\)。如果服务器验证资源的ETag没有改变（该资源没有更新），将返回一个304状态告诉客户端使用本地缓存文件。否则将返回200状态和新的资源和Etag.  使用这样的机制将提高网站的性能。例如: If-None-Match: "03f2b33c0bfcc1:0"。

![](/assets/121800390202646.png)

Pragma：指定“no-cache”值表示服务器必须返回一个刷新后的文档，即使它是代理服务器而且已经有了页面的本地拷贝；在HTTP/1.1版本中，它和Cache-Control:no-cache作用一模一样。Pargma只有一个用法， 例如： Pragma: no-cache  
注意: 在HTTP/1.0版本中，只实现了Pragema:no-cache, 没有实现Cache-Control

Cache-Control：指定请求和响应遵循的缓存机制。缓存指令是单向的（响应中出现的缓存指令在请求中未必会出现），且是独立的（在请求消息或响应消息中设置Cache-Control并不会修改另一个消息处理过程中的缓存处理过程）。请求时的缓存指令包括no-cache、no-store、max-age、max-stale、min-fresh、only-if-cached，响应消息中的指令包括public、private、no-cache、no-store、no-transform、must-revalidate、proxy-revalidate、max-age、s-maxage。

Cache-Control:Public 可以被任何缓存所缓存  
Cache-Control:Private 内容只缓存到私有缓存中  
Cache-Control:no-cache 所有内容都不会被缓存  
Cache-Control:no-store 用于防止重要的信息被无意的发布。在请求消息中发送将使得请求和响应消息都不使用缓存。  
Cache-Control:max-age 指示客户机可以接收生存期不大于指定时间（以秒为单位）的响应。  
Cache-Control:min-fresh 指示客户机可以接收响应时间小于当前时间加上指定时间的响应。  
Cache-Control:max-stale 指示客户机可以接收超出超时期间的响应消息。如果指定max-stale消息的值，那么客户机可以接收超出超时期指定值之内的响应消息。

Accept：浏览器端可以接受的MIME类型。例如：Accept: text/html 代表浏览器可以接受服务器回发的类型为 text/html 也就是我们常说的html文档，如果服务器无法返回text/html类型的数据，服务器应该返回一个406错误\(non acceptable\)。通配符 \* 代表任意类型，例如 Accept: \*/\* 代表浏览器可以处理所有类型，\(一般浏览器发给服务器都是发这个\)。

Accept-Encoding：浏览器申明自己可接收的编码方法，通常指定压缩方法，是否支持压缩，支持什么压缩方法（gzip，deflate）;Servlet能够向支持gzip的浏览器返回经gzip编码的HTML页面。许多情形下这可以减少5到10倍的下载时间。例如： Accept-Encoding: gzip, deflate。如果请求消息中没有设置这个域，服务器假定客户端对各种内容编码都可以接受。

Accept-Language：浏览器申明自己接收的语言。语言跟字符集的区别：中文是语言，中文有多种字符集，比如big5，gb2312，gbk等等；例如：Accept-Language: en-us。如果请求消息中没有设置这个报头域，服务器假定客户端对各种语言都可以接受。

Accept-Charset：浏览器可接受的字符集。如果在请求消息中没有设置这个域，缺省表示任何字符集都可以接受。

User-Agent：告诉HTTP服务器，客户端使用的操作系统和浏览器的名称和版本。  
例如： User-Agent: Mozilla/4.0 \(compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; CIBA; .NET CLR 2.0.50727; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729; .NET4.0C; InfoPath.2; .NET4.0E\)。

Content-Type：例如：Content-Type: application/x-www-form-urlencoded。

Referer：包含一个URL，用户从该URL代表的页面出发访问当前请求的页面。提供了Request的上下文信息的服务器，告诉服务器我是从哪个链接过来的，比如从我主页上链接到一个朋友那里，他的服务器就能够从HTTP Referer中统计出每天有多少用户点击我主页上的链接访问他的网站。  
例如: Referer:[http://translate.google.cn/?hl=zh-cn&tab=wT](http://translate.google.cn/?hl=zh-cn&tab=wT)

Connection：  
例如：Connection: keep-alive 当一个网页打开完成后，客户端和服务器之间用于传输HTTP数据的TCP连接不会关闭，如果客户端再次访问这个服务器上的网页，会继续使用这一条已经建立的连接。HTTP 1.1默认进行持久连接。利用持久连接的优点，当页面包含多个元素时（例如Applet，图片），显著地减少下载所需要的时间。要实现这一点，Servlet需要在应答中发送一个Content-Length头，最简单的实现方法是：先把内容写入ByteArrayOutputStream，然后在正式写出内容之前计算它的大小。  
Connection: close 代表一个Request完成后，客户端和服务器之间用于传输HTTP数据的TCP连接会关闭，当客户端再次发送Request，需要重新建立TCP连接。

Host：（发送请求时，该头域是必需的）主要用于指定被请求资源的Internet主机和端口号，它通常从HTTP URL中提取出来的。HTTP/1.1请求必须包含主机头域，否则系统会以400状态码返回。  
例如: 我们在浏览器中输入：[http://www.guet.edu.cn/index.html，浏览器发送的请求消息中，就会包含Host请求头域：Host：http://www.guet.edu.cn，此处使用缺省端口号80，若指定了端口号，则变成：Host：指定端口号。](http://www.guet.edu.cn/index.html，浏览器发送的请求消息中，就会包含Host请求头域：Host：http://www.guet.edu.cn，此处使用缺省端口号80，若指定了端口号，则变成：Host：指定端口号。)

Cookie：最重要的请求头之一, 将cookie的值发送给HTTP服务器。

Content-Length：表示请求消息正文的长度。例如：Content-Length: 38。

Authorization：授权信息，通常出现在对服务器发送的WWW-Authenticate头的应答中。主要用于证明客户端有权查看某个资源。当浏览器访问一个页面时，如果收到服务器的响应代码为401（未授权），可以发送一个包含Authorization请求报头域的请求，要求服务器对其进行验证。

UA-Pixels，UA-Color，UA-OS，UA-CPU：由某些版本的IE浏览器所发送的非标准的请求头，表示屏幕大小、颜色深度、操作系统和CPU类型。

From：请求发送者的email地址，由一些特殊的Web客户程序使用，浏览器不会用到它。

Range：可以请求实体的一个或者多个子范围。例如，  
表示头500个字节：bytes=0-499  
表示第二个500字节：bytes=500-999  
表示最后500个字节：bytes=-500  
表示500字节以后的范围：bytes=500-  
第一个和最后一个字节：bytes=0-0,-1  
同时指定几个范围：bytes=500-600,601-999  
但是服务器可以忽略此请求头，如果无条件GET包含Range请求头，响应会以状态码206（PartialContent）返回而不是以200（OK）。

_**6.6、HTTP常见的响应头**_

Allow：服务器支持哪些请求方法（如GET、POST等）。

Date：表示消息发送的时间，时间的描述格式由rfc822定义。例如，Date:Mon,31Dec200104:25:57GMT。Date描述的时间表示世界标准时，换算成本地时间，需要知道用户所在的时区。你可以用setDateHeader来设置这个头以避免转换时间格式的麻烦

Expires：指明应该在什么时候认为文档已经过期，从而不再缓存它，重新从服务器获取，会更新缓存。过期之前使用本地缓存。HTTP1.1的客户端和缓存会将非法的日期格式（包括0）看作已经过期。eg：为了让浏览器不要缓存页面，我们也可以将Expires实体报头域，设置为0。  
例如: Expires: Tue, 08 Feb 2022 11:35:14 GMT

P3P：用于跨域设置Cookie, 这样可以解决iframe跨域访问cookie的问题  
例如: P3P: CP=CURa ADMa DEVa PSAo PSDo OUR BUS UNI PUR INT DEM STA PRE COM NAV OTC NOI DSP COR

Set-Cookie：非常重要的header, 用于把cookie发送到客户端浏览器，每一个写入cookie都会生成一个Set-Cookie。  
例如: Set-Cookie: sc=4c31523a; path=/; domain=.acookie.taobao.com

ETag：和If-None-Match 配合使用。

Last-Modified：用于指示资源的最后修改日期和时间。Last-Modified也可用setDateHeader方法来设置。

Content-Type：WEB服务器告诉浏览器自己响应的对象的类型和字符集。Servlet默认为text/plain，但通常需要显式地指定为text/html。由于经常要设置Content-Type，因此HttpServletResponse提供了一个专用的方法setContentType。可在web.xml文件中配置扩展名和MIME类型的对应关系。  
例如:Content-Type: text/html;charset=utf-8  
　　 Content-Type:text/html;charset=GB2312  
　　 Content-Type: image/jpeg

媒体类型的格式为：大类/小类，比如text/html。  
IANA\(The Internet Assigned Numbers Authority，互联网数字分配机构\)定义了8个大类的媒体类型，分别是:  
application— \(比如: application/vnd.ms-excel.\)  
audio \(比如: audio/mpeg.\)  
image \(比如: image/png.\)  
message \(比如,:message/http.\)  
model\(比如:model/vrml.\)  
multipart \(比如:multipart/form-data.\)  
text\(比如:text/html.\)  
video\(比如:video/quicktime.\)

Content-Range：用于指定整个实体中的一部分的插入位置，他也指示了整个实体的长度。在服务器向客户返回一个部分响应，它必须描述响应覆盖的范围和整个实体长度。一般格式：Content-Range:bytes-unitSPfirst-byte-pos-last-byte-pos/entity-length。  
例如，传送头500个字节次字段的形式：Content-Range:bytes0-499/1234如果一个http消息包含此节（例如，对范围请求的响 应或对一系列范围的重叠请求），Content-Range表示传送的范围。

Content-Length：指明实体正文的长度，以字节方式存储的十进制数字来表示。在数据下行的过程中，Content-Length的方式要预先在服务器中缓存所有数据，然后所有数据再一股脑儿地发给客户端。只有当浏览器使用持久HTTP连接时才需要这个数据。如果你想要利用持久连接的优势，可以把输出文档写入ByteArrayOutputStram，完成后查看其大小，然后把该值放入Content-Length头，最后通过byteArrayStream.writeTo\(response.getOutputStream\(\)发送内容。  
例如: Content-Length: 19847

Content-Encoding：WEB服务器表明自己使用了什么压缩方法（gzip，deflate）压缩响应中的对象。只有在解码之后才可以得到Content-Type头指定的内容类型。利用gzip压缩文档能够显著地减少HTML文档的下载时间。Java的GZIPOutputStream可以很方便地进行gzip压缩，但只有Unix上的Netscape和Windows上的IE 4、IE 5才支持它。因此，Servlet应该通过查看Accept-Encoding头（即request.getHeader\("Accept-Encoding"\)）检查浏览器是否支持gzip，为支持gzip的浏览器返回经gzip压缩的HTML页面，为其他浏览器返回普通页面。  
例如：Content-Encoding：gzip

Content-Language：WEB服务器告诉浏览器自己响应的对象所用的自然语言。例如： Content-Language:da。没有设置该域则认为实体内容将提供给所有的语言阅读。

Server：指明HTTP服务器用来处理请求的软件信息。例如：Server: Microsoft-IIS/7.5、Server：Apache-Coyote/1.1。此域能包含多个产品标识和注释，产品标识一般按照重要性排序。

X-AspNet-Version：如果网站是用ASP.NET开发的，这个header用来表示ASP.NET的版本。  
例如: X-AspNet-Version: 4.0.30319

X-Powered-By：表示网站是用什么技术开发的。  
例如： X-Powered-By: ASP.NET

Connection：  
例如：Connection: keep-alive 当一个网页打开完成后，客户端和服务器之间用于传输HTTP数据的TCP连接不会关闭，如果客户端再次访问这个服务器上的网页，会继续使用这一条已经建立的连接。  
Connection: close 代表一个Request完成后，客户端和服务器之间用于传输HTTP数据的TCP连接会关闭，当客户端再次发送Request，需要重新建立TCP连接。

Location：用于重定向一个新的位置，包含新的URL地址。表示客户应当到哪里去提取文档。Location通常不是直接设置的，而是通过HttpServletResponse的sendRedirect方法，该方法同时设置状态代码为302。Location响应报头域常用在更换域名的时候。

Refresh：表示浏览器应该在多少时间之后刷新文档，以秒计。除了刷新当前文档之外，你还可以通过setHeader\("Refresh", "5; URL=\[[http://host/path"\)让浏览器读取指定的页面。注意这种功能通常是通过设置HTML页面HEAD区的&lt;META\]\(http://host/path"\)让浏览器读取指定的页面。注意这种功能通常是通过设置HTML页面HEAD区的](http://host/path"%29让浏览器读取指定的页面。注意这种功能通常是通过设置HTML页面HEAD区的<META]%28http://host/path"%29让浏览器读取指定的页面。注意这种功能通常是通过设置HTML页面HEAD区的)&lt;META\) HTTP-EQUIV="Refresh" CONTENT="5;URL=\[[http://host/path"&gt;实现，这是因为，自动刷新或重定向对于那些不能使用CGI或Servlet的HTML编写者十分重要。但是，对于Servlet来说，直接设置Refresh头更加方便。注意Refresh的意义是“N秒之后刷新本页面或访问指定页面”，而不是“每隔N秒刷新本页面或访问指定页面”。因此，连续刷新要求每次都发送一个Refresh头，而发送204状态代码则可以阻止浏览器继续刷新，不管是使用Refresh头还是&lt;META\]\(http://host/path"&gt;实现，这是因为，自动刷新或重定向对于那些不能使用CGI或Servlet的HTML编写者十分重要。但是，对于Servlet来说，直接设置Refresh头更加方便。注意Refresh的意义是“N秒之后刷新本页面或访问指定页面”，而不是“每隔N秒刷新本页面或访问指定页面”。因此，连续刷新要求每次都发送一个Refresh头，而发送204状态代码则可以阻止浏览器继续刷新，不管是使用Refresh头还是](http://host/path">实现，这是因为，自动刷新或重定向对于那些不能使用CGI或Servlet的HTML编写者十分重要。但是，对于Servlet来说，直接设置Refresh头更加方便。注意Refresh的意义是“N秒之后刷新本页面或访问指定页面”，而不是“每隔N秒刷新本页面或访问指定页面”。因此，连续刷新要求每次都发送一个Refresh头，而发送204状态代码则可以阻止浏览器继续刷新，不管是使用Refresh头还是<META]%28http://host/path">实现，这是因为，自动刷新或重定向对于那些不能使用CGI或Servlet的HTML编写者十分重要。但是，对于Servlet来说，直接设置Refresh头更加方便。注意Refresh的意义是“N秒之后刷新本页面或访问指定页面”，而不是“每隔N秒刷新本页面或访问指定页面”。因此，连续刷新要求每次都发送一个Refresh头，而发送204状态代码则可以阻止浏览器继续刷新，不管是使用Refresh头还是)&lt;META\) HTTP-EQUIV="Refresh" ...&gt;。注意Refresh头不属于HTTP 1.1正式规范的一部分，而是一个扩展，但Netscape和IE都支持它。

WWW-Authenticate：该响应报头域必须被包含在401（未授权的）响应消息中，客户端收到401响应消息时候，并发送Authorization报头域请求服务器对其进行验证时，服务端响应报头就包含该报头域。  
eg：WWW-Authenticate:Basic realm="Basic Auth Test!" //可以看出服务器对请求资源采用的是基本验证机制。

**七、解决HTTP无状态的问题**

_**7.1、通过Cookies保存状态信息**_  
通过Cookies，服务器就可以清楚的知道请求2和请求1来自同一个客户端。

![](/assets/122123269892896.png)
7.2、通过Session保存状态信息

Session机制是一种服务器端的机制，服务器使用一种类似于散列表的结构（也可能就是使用散列表）来保存信息。
当程序需要为某个客户端的请求创建一个session的时候，服务器首先检查这个客户端的请求里是否已包含了一个session标识 - 称为 session id，如果已包含一个session id则说明以前已经为此客户端创建过session，服务器就按照session id把这个 session检索出来使用（如果检索不到，可能会新建一个），如果客户端请求不包含session id，则为此客户端创建一个session并且生成一个与此session相关联的session id，session id的值应该是一个既不会重复，又不容易被找到规律以仿造的字符串，这个session id将被在本次响应中返回给客户端保存。

Session的实现方式：

1、使用Cookie来实现
服务器给每个Session分配一个唯一的JSESSIONID，并通过Cookie发送给客户端。
当客户端发起新的请求的时候，将在Cookie头中携带这个JSESSIONID。这样服务器能够找到这个客户端对应的Session。


























