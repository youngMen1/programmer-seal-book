**1. OSI七层和TCP/IP四层的关系**

1.1 OSI引入了服务、接口、协议、分层的概念，TCP/IP借鉴了OSI的这些概念建立TCP/IP模型。

1.2 OSI先有模型，后有协议，先有标准，后进行实践；而TCP/IP则相反，先有协议和应用再提出了模型，且是参照的OSI模型。

1.3 OSI是一种理论下的模型，而TCP/IP已被广泛使用，成为网络互联事实上的标准。

TCP：transmission control protocol 传输控制协议

UDP：user data protocol 用户数据报协议

| OSI七层网络模型 | TCP/IP四层概念模型 | 对应网络协议 |
| :--- | :--- | :--- |
| 应用层（Application） | 应用层 | HTTP、TFTP, FTP, NFS, WAIS、SMTP |
| 表示层（Presentation） |  | Telnet, Rlogin, SNMP, Gopher |
| 会话层（Session） | 会话层（Session） | SMTP, DNS |
| 传输层（Transport） | 传输层 | TCP, UDP |
| 网络层（Network） | 网络层 | IP, ICMP, ARP, RARP, AKP, UUCP |
| 数据链路层（Data Link） | 数据链路层 | FDDI, Ethernet, Arpanet, PDN, SLIP, PPP |
|  | 物理层（Physical） | IEEE 802.1A, IEEE 802.2到IEEE 802.11 |



