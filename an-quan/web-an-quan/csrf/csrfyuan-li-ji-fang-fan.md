## [CSRF原理及防范](https://coderxing.gitbooks.io/architecture-evolution/di-san-pian-ff1a-bu-luo/641-web-an-quan-fang-fan/6412-csrf.html)

CSRF \(Cross—Site Request Forgery\)，既跨站点请求伪造，也被叫做 XSRF，和 XSS 一样也是一种比较常见的 web 攻击。CSRF攻击者会过过构造的第三方页面诱导受害者完成加载或者点击，利用受害者的权限，以其身份向合法网站发起恶意请求，通常用户发生状态改变的请求，比如虚拟货币的转账，账号信息修改，恶意发邮件等等，由于具有一定的隐蔽性，所以比较防范。
![img](/staic/image/0D485010-0672-43D8-AC1E-9468D64AA14F.png)



