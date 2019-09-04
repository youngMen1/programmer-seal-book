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

