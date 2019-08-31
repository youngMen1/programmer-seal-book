## [CSRF原理及防范](https://coderxing.gitbooks.io/architecture-evolution/di-san-pian-ff1a-bu-luo/641-web-an-quan-fang-fan/6412-csrf.html)

CSRF \(Cross—Site Request Forgery\)，既跨站点请求伪造，也被叫做 XSRF，和 XSS 一样也是一种比较常见的 web 攻击。CSRF攻击者会过过构造的第三方页面诱导受害者完成加载或者点击，利用受害者的权限，以其身份向合法网站发起恶意请求，通常用户发生状态改变的请求，比如虚拟货币的转账，账号信息修改，恶意发邮件等等，由于具有一定的隐蔽性，所以比较防范。
![img](/staic/image/0D485010-0672-43D8-AC1E-9468D64AA14F.png)

我们通过一个简单例子来演示 CSRF的攻击原理，如上图所示（TODO需要重做），正常网站A，攻击网站B，用户C，正常网站具有一个虚拟币转账的公功能，转账通过GET请求完成，且网站A不具备 CSRF 防范机制，攻击者利用这个漏洞恶意构造网站B，诱导用户C访问，达到恶意转账目的。
转账的GET请求如下：


```
GET http://normal-site.com/transfer.do?from=rommel&to=alice&amount=100 HTTP/1.1

```
这是我们出于演示方便使用GET请求来完成转账，实际应用远比这个列子要复杂，需要更加安全的机制，比如一次性 token，https 等，而且一般都是用POST请求。



