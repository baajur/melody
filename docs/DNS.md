# DNS
参考 https://draveness.me/whys-the-design-dns-udp-tcp
## 概述
在绝大多数情况下，DNS 都是使用 UDP 协议进行通信的，DNS 协议在设计之初也推荐我们在进行域名解析时首先使用 UDP，这确实能解决很多需求，但是不能解决全部的问题。

实际上，DNS 不仅使用了 UDP 协议，也使用了 TCP 协议

对 DNS 协议进行简单的介绍：DNS 查询的类型不止包含 A 记录、CNAME 记录等常见查询，还包含 AXFR 类型的特殊查询，这种特殊查询主要用于 DNS 区域传输，它的作用就是在多个命名服务器之间快速迁移记录，由于查询返回的响应比较大，所以会使用 TCP 协议来传输数据包。

区域传输：DNS 主从复制，就是将主 DNS 服务器的解析库复制传送至从 DNS 服务器，进而从服务器就可以进行正向、反向解析了。从服务器向主服务器查询更新数据，保证数据一致性，此为区域传送。也可以说，DNS 区域传输，就是 DNS 主从复制的实现方法，DNS 主从复制是 DNS 区域传输的表现形式。

RFC 文档中与 UDP/TCP 协议相关内容的总结：

DNS 在被设计之初就可以同时使用 TCP 和 UDP 协议，对于绝大多数的 DNS 查询来说都会使用 UDP 数据报进行传输，TCP 协议只会在区域传输的场景中使用，其中 UDP 数据包只会传输最大 512 字节的数据，多余的会被截断；

RFC1123 预测了 DNS 记录中存储的数据会越来越多，同时也第一次显式的指出了发现 UDP 包被截断时应该通过 TCP 协议重试。

由于互联网的发展，人们发现 IPv4 已经不够分配了，所以引入了更长的 IPv6，DNS 也进行了协议上的支持，随后又增加了在鉴权和安全方面的支持，但是也带来了巨大的 DNS 记录，UDP 数据包被截断变得非常常见。

RFC7766 才终于提出了使用 TCP 协议作为主要协议来解决 UDP 无法解决的问题，TCP 协议也不再只是一种重试时使用的机制，随后出现的 DNS over TLS 和 DNS over HTTP 也都是对 DNS 协议的一种补充。

## 为什么小数据包用 UDP？

以前的 DNS 查询的数据包较小、机制简单；

UDP 协议的额外开销小、有着更好的性能表现，而 TCP 需要三次握手，相对于当时的小数据包来说是很大的开销；

## 为什么大数据包用 TCP？

DNS 查询由于 DNSSEC 和 IPv6 的引入迅速膨胀，导致 DNS 响应经常超过 MTU（传送链路的最大传输单元，也就是单个数据包大小的上限，一般为 1500 字节） 造成数据的分片和丢失，我们需要依靠更加可靠的 TCP 协议（通过序列号、重传等机制能够保证消息的不重不漏，消息接受方的 TCP 栈会对分片的数据重新进行拼装，DNS 等应用层协议可以直接使用处理好的完整数据）完成数据的传输；

随着 DNS 查询中包含的数据不断增加，TCP 协议头以及三次握手带来的额外开销比例逐渐降低，不再是占据总传输数据大小的主要部分；
