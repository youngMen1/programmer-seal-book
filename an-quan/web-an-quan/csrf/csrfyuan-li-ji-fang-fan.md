## [CSRF原理及防范](https://coderxing.gitbooks.io/architecture-evolution/di-san-pian-ff1a-bu-luo/641-web-an-quan-fang-fan/6412-csrf.html)

CSRF \(Cross—Site Request Forgery\)，既跨站点请求伪造，也被叫做 XSRF，和 XSS 一样也是一种比较常见的 web 攻击。CSRF攻击者会过过构造的第三方页面诱导受害者完成加载或者点击，利用受害者的权限，以其身份向合法网站发起恶意请求，通常用户发生状态改变的请求，比如虚拟货币的转账，账号信息修改，恶意发邮件等等，由于具有一定的隐蔽性，所以比较防范。  
![img](/staic/image/0D485010-0672-43D8-AC1E-9468D64AA14F.png)

我们通过一个简单例子来演示 CSRF的攻击原理，如上图所示（TODO需要重做），正常网站A，攻击网站B，用户C，正常网站具有一个虚拟币转账的公功能，转账通过GET请求完成，且网站A不具备 CSRF 防范机制，攻击者利用这个漏洞恶意构造网站B，诱导用户C访问，达到恶意转账目的。  
转账的GET请求如下：

```
GET http://normal-site.com/transfer.do?from=rommel&to=alice&amount=100 HTTP/1.1
```

这是我们出于演示方便使用GET请求来完成转账，实际应用远比这个列子要复杂，需要更加安全的机制，比如一次性 token，https 等，而且一般都是用POST请求。  
CSRF 的攻击过程过程图上图所示：  
CSRF 攻击有一个前提条件，是用户具有某个正常访问的访问权限。一般网站的访问断线都具备一定的有效期，比如1天过期，或者几个小时骨气，再次期间权限信息会保留在用户浏览器的cookie中，这本例子中假设用户C刚刚登录了网站A，全新还没有过期。  
攻击者利用正常网站A的CSFR漏洞，构造页面一个恶意网页B，在页面中包含对发往正常网站A的请求，在用户C加载页面B（或者点击某些元素时触发）时，会触发攻击请求，目的是为了实现虚拟币的转账，请求可能隐藏得很深，用户并不一定能发现，伪造的请求如下：

```
<img widht=0 height=0 src="http://normal-site.com/transfer.do?from=rommel&to=attacker&amount=100" />
```

由于加载恶意页面B和触发攻击请求都是在用户浏览器端完成的，因为之前用户登录过正常网站，发往正常网站的请求会带有用户授权信息（在cookie中），在授权信息没有过期的情况达到攻击目的。

## CSRF 的防范

​ 目前主要有如下几种方式：  
​添加 Referer 域名白名单：HTTP 的 Referer 头记录了当前请求的来源页面的URL，如果用户是通过浏览器打开的网页一般都会带有这个信息。可以验证 URL 的域名是否在网站允许的白名单呢内，如果不在则拒绝请求。这种方式实现比较简单，而且可以在web 服务器层统一配置，减少了后端开发成本，当 Referer 页可以伪造，用户浏览器的可靠性也不能完全信赖，判断 Referer 可以做为一种辅助手段，但不能根治 CSRF。  
令牌 \(Token \)验证：令牌验证的方式，这是目前方法CSRF的一种普遍方法，其原理是在用户正式提交数据更新之前，给用户生成一个 token，一方面token保存在服务端，比如Session 或者缓存中，一方面输出请求发生的页面上，在用户提交请求时连同token一同提交，服务获得接收到请求之后再做token验证，token 不存在、或者token不一致，或者失效都算作验证失败。token 的生成具有一定的随机性，而且是在提交数据的页面生成，攻击这往往难于伪造。token 一般作为一个post 字段提交，或者作为ajax请求的header信息提交，例如：

```
<form>
<input name="from" value="rommel" />
<input name="to" value="" />
<input name="csrf_token" value="QMYjiBlZ9V9mGnap" />
</form>
$.ajax({
headers: {
"X-CSRF-TOKEN": "QMYjiBlZ9V9mGnap"
}
});
```

```
另外，为了更可靠的安全性，token 在使用之后一般置为失效，避免被窃取。<script src="https://cdn.bootcss.com/jquery/3.2.1/jquery.min.js"></script>
<script type="text/javascript">
$.ajax({
url:"http://www.a.com/jsonp.php?callback=cb",
dataType:'jsonp',
jsonp:'callback',
success:function(result) {
console.log(result);
}
});
</script>
二次验证：对于一些铭感操作，比如设计到交易的操作，可以更加严格一些。在用户提交时可以让用户输入验证码，或者再次输入交易密码，确保是用户的真实操作，而不是机器触发的。
使用 SameSite Cookie 属性：SameSite 是在跨域请求时是否传输 Cookie 的一种约束，顾名思义在 SameSite 的限制下，Cookie 的传输必须在同一个域名下。SameSite 属性有两种限制模式 strict 和 lax，默认为 lax。下面举例说明，对于下面的代码：
```

```
<?php
//www.a.com/test_samecookie.php
//第二个参数为false表示不不进行覆盖，否则只能设置一个cookie
header("Set-Cookie: cookie1=value1; domain=a.com; SameSite=Strict", false);
header("Set-Cookie: cookie2=value2; domain=a.com; SameSite=Lax", false);
?>
```

访问，www.a.com/test\_samecookie.php 通过Chome 的开发者工具查看cookie信息，如下：  
![img](/static/image/3E9E901F-40EE-401E-B3ED-C36BFAC4CFE5.png)  
在 www.b.com/access\_same\_page\_in\_site.php 构造同样的页面访问 www.a.com 中的任何页面。

```
<a href="www.a.com/test.html">test</a>
```

点击页面上的链接之后，会发现，只有 cookie2 传递到了www.a.com strict 模式限制所有的跨域cookie传输，lax 模式相当于在安全性和可用性之间做了个折中，某些场景允许跨域传输，比如 a 标签中中的GET请求，但会对 img、iframe 和 ajax 中的GET请求以及POST请求做限制，

```
目前只有 Chrome 和 Opera 浏览器支持 SameSite 属性。
<?php
$callback = $_GET['callback'];
$data = array(
'userid' => '12345678',
'usernick' => 'coderxing'
);
$json = json_encode($data);
echo "$callback($json);";
?>
```

输出内容为：

```
cb({"userid":"12345678","usernick":"coderxing"});
```

B 网站通过 JQuery 使用 jsonp 的代码如下：

```
<script src="https://cdn.bootcss.com/jquery/3.2.1/jquery.min.js"></script>
<script type="text/javascript">
$.ajax({
url:"http://www.a.com/jsonp.php?callback=cb",
dataType:'jsonp',
jsonp:'callback',
success:function(result) {
console.log(result);
}
});
</script>
```

通过Chrome浏览器访问：

[http://www.b.com/use\_jsonp.html，控制台会输出](http://www.b.com/use_jsonp.html%EF%BC%8C%E6%8E%A7%E5%88%B6%E5%8F%B0%E4%BC%9A%E8%BE%93%E5%87%BA):

```
AB3E8ED3-5F75-4F76-B0E1-1413742BC7D6.png
```



