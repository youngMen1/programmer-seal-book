## sql注入

SQL 注入页也是一种常见的 Web 攻击方式，主要是利用后端程序的漏洞，针对数据库（主要是关系型数据库）进行攻击。攻击者通常通过输入精心构造的参数，绕过服务器端的验证，来执行恶意 SQL 语句，SQL注入会造成拖库（原始数据表被攻击者获取），绕过权限验证，或者串改、破坏、删除数据。数据往往是一个网站甚至一个公司的生命，一但SQL漏洞被攻击者利用通常会产生非常严重的后果，对于SQL 注入的防范需要开发者倍加重视。下面通过一个例子来演示SQL注入的攻击过程，对于一个登陆验证的请求一般需要通过用户输入的用户名和密码进行查询验证，查询SQL 语句如下：

```
SELECT * FROM users WHERE username = "$username" AND password = "$password";
```

比如攻击者已知一个用户名archer2017，可以构造密码为 anywords" OR 1=1，此时，此时后端程序的执行的SQL语句为：

```
SELECT * FROM users WHERE username = "archer2017" AND password = "anywords" OR 1=1;
```

这样无论输入什么样的密码，都会要绕过验证，如果更验证一些，构造密码为 anywords" OR 1=1;DROP TABLE users ，那么整个表都将被删除，执行SQL如下:

```
SELECT * FROM users WHERE username = "archer2017" AND password = "anywords" OR 1=1;DROP TABLE users;
```

##### SQL 注入的防范 {#sql-注入的防范}

​ SQL 注入发生绝大多数情况都是直接使用户输入参数拼装 SQL 语句造成的，防范 SQL 注入，主要的方式丢失避免直接使用用户输入的数据。

​ 使用预编译语句（PreparedStatement）：一方面可以加速 SQL 的执行，一方面可以防止SQL注入。理解 PreparedStatement 防范 SQL 注入的原理，先要理解预编译语句的原理，如下图所示：

4C186014-E51E-43B8-86FC-21092D2C30D2.png

