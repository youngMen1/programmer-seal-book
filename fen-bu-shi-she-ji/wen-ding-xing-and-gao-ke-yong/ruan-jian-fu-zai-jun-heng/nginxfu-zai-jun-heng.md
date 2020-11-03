# 1.Nginx负载均衡
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

，该组中包含三台主机。当用户通过 HTTP 访问 www.example.com 域名时，请求会按照轮询（**轮询是默认的转发策略**）的方式转到 app\_group 对应的后端服务器上。

Nginx 中使用负载均衡功能需要依赖 ngx\_http\_upstream\_module 模块。可以通过

`nginx -V`

查看已安装的模块。

## 1.1.Nginx 自带多种转发策略：

* **轮询（Round-Robin ）策略**，这是也是默认的策略，Nginx 会根据服务器配置列表将机请求轮流转发到后端服务器上。

* **最少连接（Least-connected）策略**，在该策略下 Nginx 会尝试避繁忙的机器，将请转到有减少连接较少的机器上，该策略可以环节呢后端服务器负载冷热不均的问题，使用`least_conn`配置该策略，例如：

```
upstream app_group {
          least_conn;
        server 192.168.56.102;
        server 192.168.56.103;
        server 192.168.56.104;
    }
```

**最少耗时（Least-time）策略**：该策略会将请求转到都具有最小平均影响时间和最少活动链接的后端服务器，使用`least_time`

配置该策略，例如：

```
upstream app_group {
          least_time;
        server 192.168.56.102;
        server 192.168.56.103;
        server 192.168.56.104;
    }
```

**回话保持（Session persistence）策略**，在轮询或者最少连接策略下，同一个用户（同一个客户端IP）的不同请求会随机分配到不同的后端服务器上。回话保持策略会通过客户端 IP 计算一个成 Hash 值，按照 hash 值分配后端服务器，该策略可以使得同一个用户的请求落在同一个后端主机上。使用`ip_hash`配置该指令，例如：

```
 upstream app_group {
          ip_hash;
        server 192.168.56.102;
        server 192.168.56.103;
        server 192.168.56.104;
    }
```

**自定义 Hash 策略**：前面介绍的回话保持策略通过客户端IP计算Hash只，自定义 Hash 策略可以根据用户自己定的主键计算hash值，比如使用 uri 作为 key 计算hash，例如：

```
upstream app_group {
          hash $request_uri consistent;
        server 192.168.56.102;
        server 192.168.56.103;
        server 192.168.56.104;
    }
```

其中的 consistent 是可选参数，表示使用一致性 hash 算法（我们会在后面的章节介绍一致性Hash 算法 TODO），该算法客可以环节由于后端主机的添加或减少带来的映射关系的变化。

* **基于权重策略**：Nginx 可以为不同的主机配置不同的权重，按照权重比例转发流量，比如下面的配置会按照 3:1:1 的比例分配流量：

```
 upstream app_group {
        server 192.168.56.102 weight=3;
        server 192.168.56.103;
        server 192.168.56.104;
    }
```

前文介绍的轮询策略，可以理解为按照 1:1:1 的比例分配流量。

​ 除了按照IP地址指定主机意外，server 指令支持多种配置形式，可以按照域名进行配置，也可以按照unix 套接字指定，也可以指定端口号，例如：

```
 upstream app_group {
        server vm1.example.com
        server 192.168.56.103:80;
        server 192.168.56.104:8080;
          server unix:/tmp/backend.socks;
    }
```

Server 指令常用的参数如下：

* weight ： 服务器权重，默认为1。

* max\_conns：限制单个服务器同时最多的连接数，默认为 0，表示不限制。

* max\_fails：在 fail\_timeout 给定时间范围内，尝试和后端主机通信的最大连续失败次数，默认为1次，0 表示不对该主机进行健康检查。当达到该值时，对应主机被置为不可用，请求将不会被转发到不可用的主机上。在下一个 fail\_timeout 时间只有，nginx 会尝试转发请求到不可用主机上，如果请求成功，则原来不可用的主机被重新置为可用。

* fail\_timeout：定义服务不可以用的时间段，以及定义最大尝试失败次数的时间周期，默认为10秒。

* backup：标记当前主机为后备服务器，当其他服务不可都不可用时，请求才会转发给后备主机。

* down：永久标记当前主机不可用。

* slow\_start：设置慢启动时间，当前服务器从不可用变为可用，或者权重从0变为设定值时，等在一段时间再完全恢复。 有时后端服务完全启动需要一段时间，某条请求的转发成功并不一定能完全代表服务已经完全启动，该值可以用户延迟恢复，等待后端服务器的完全启动。（是不是商用版才有？）

* service，型

* Route

* queue 商用版（才有？）

  ​

​ 社区版的 Nginx 靠 max\_fails 和 fail\_timeout 进行监控检查。Nginx 的商用版提供更加强大的监控检查机制。

主动健康检查:

​ 在 Nginx 商用版本（Nginx Plus）中，可以定期主动发送健康检查请求，通过后端应用程序的响应结果来判断服务是否可用，开启主动健康检查需要在 location 块中包含 health\_check 指令，同时还用使用 zone 指定定义共享内存区（在Nginx多个worker进程中共享），用来记录中间状态，例如：

```
http {
    upstream app_group {
        zone backend 64k;
        server 192.168.56.102;
        server 192.168.56.103;
        server 192.168.56.104;
    }

    server {
        listen 80;
        server_name  www.example.com;

        location / {
              health_check interval=5s uri=/test.od match=statusok;
            proxy_pass http://app_group;
        }
        match statusok {
            # Used for /test.php health check
            status 200;
            header Content-Type = text/html;
            body ~ "Server[0-9]+ is alive";
        }
    }
}
```



