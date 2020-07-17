# 1.MySQL命令详解


```
show processlist;
```


![](/static/image/微信截图_20200717173429.png)

SHOW PROCESSLIST显示哪些线程正在运行

如果您有root权限，您可以看到所有线程。否则，您只能看到登录的用户自己的线程，通常只会显示100条如果想看跟多的可以使用full修饰（show full processlist）

说明各列的含义和用途:
 id:       #ID标识，要kill一个语句的时候很有用
use:      #当前连接用户
host:     #显示这个连接从哪个ip的哪个端口上发出
db:       #数据库名
command:  #连接状态，一般是休眠（sleep），查询（query），连接（connect）
time:     #连接持续时间，单位是秒
state:    #显示当前sql语句的状态
info:     #显示这个sql语句
