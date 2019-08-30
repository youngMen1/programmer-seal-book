HTTP 2.0是在SPDY（An experimental protocol for a faster web, The Chromium Projects）基础上形成的下一代互联网通信协议。HTTP/2 的目的是通过支持请求与响应的多路复用来较少延迟，通过压缩HTTPS首部字段将协议开销降低，同时增加请求优先级和服务器端推送的支持。

本文目的是学习HTTP 2.0的原理并研究其通信的详细细节。大部分知识点源于《Web性能权威指南》。



