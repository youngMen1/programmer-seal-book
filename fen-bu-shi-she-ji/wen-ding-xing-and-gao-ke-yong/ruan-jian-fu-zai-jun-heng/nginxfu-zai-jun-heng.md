Nginx 以其优异的性能，不仅作为 web 服务器使用，也常被作为负载均衡器来使用。一台硬件负载均衡服务器动辄十几万几十万，其价格都可以购买十几台普通服务器了，当服务规模并不大时，直接采购硬件负载均衡服务器对于很多中小公司是很不划算的，所以通过 web 服务器的反向代理的方式是比较经济的方式，一般 web 服务器都有反向代理功能，Nginx 是其中典型代表。本节重点搭建Nginx 负载均衡的配置。

\(TODO 原理\)

Nginx 负载均衡的基本配置如下：

```
http {
    upstream app_group {
        server 192.168.56.102;
        server 192.168.56.103;
        server 192.168.56.104;
    }

    server {
        listen 80;
        server_name  www.example.com;

        location / {
            proxy_pass http://app_group;
        }
    }
}
```

在这个配置中，我们通过

`upstream`

定义一个后端的服务器组，命名为

`app_group`

，该组中包含三台主机。当用户通过 HTTP 访问 www.example.com 域名时，请求会按照轮询（轮询是默认的转发策略）的方式转到 app\_group 对应的后端服务器上。

Nginx 中使用负载均衡功能需要依赖 ngx\_http\_upstream\_module 模块。可以通过

`nginx -V`

查看已安装的模块。

Nginx 自带多种转发策略：

* 轮询（Round-Robin ）策略，这是也是默认的策略，Nginx 会根据服务器配置列表将机请求轮流转发到后端服务器上。

* 最少连接（Least-connected）策略，在该策略下 Nginx 会尝试避繁忙的机器，将请转到有减少连接较少的机器上，该策略可以环节呢后端服务器负载冷热不均的问题，使用`least_conn`配置该策略，例如：

```
upstream app_group {
          least_conn;
        server 192.168.56.102;
        server 192.168.56.103;
        server 192.168.56.104;
    }
```

最少耗时（Least-time）策略：该策略会将请求转到都具有最小平均影响时间和最少活动链接的后端服务器，使用

`least_time`

配置该策略，例如：

```
upstream app_group {
          least_time;
        server 192.168.56.102;
        server 192.168.56.103;
        server 192.168.56.104;
    }
```

回话保持（Session persistence）策略，在轮询或者最少连接策略下，同一个用户（同一个客户端IP）的不同请求会随机分配到不同的后端服务器上。回话保持策略会通过客户端 IP 计算一个成 Hash 值，按照 hash 值分配后端服务器，该策略可以使得同一个用户的请求落在同一个后端主机上。使用

`ip_hash`

配置该指令，例如：

```
 upstream app_group {
          ip_hash;
        server 192.168.56.102;
        server 192.168.56.103;
        server 192.168.56.104;
    }
```

自定义 Hash 策略：前面介绍的回话保持策略通过客户端IP计算Hash只，自定义 Hash 策略可以根据用户自己定的主键计算hash值，比如使用 uri 作为 key 计算hash，例如：

```
upstream app_group {
          hash $request_uri consistent;
        server 192.168.56.102;
        server 192.168.56.103;
        server 192.168.56.104;
    }
```



