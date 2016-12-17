---
title: UDP 编程
date: 2016-12-16 11:40:15
tags: QtBook
---
UDP 是 User Datagram Protocol 的简称，中文名是用户数据报协议，是一种无连接的不可靠协议，也既是说，数据发出去了，但是不能保证对方一定能接收到，就像寄信一样，虽然把信交给了邮局，中途有可能寄丢。

UDP 报文没有可靠性保证、顺序保证和流量控制字段等，可靠性较差。但是正因为 UDP 协议的控制选项较少，在数据传输过程中延迟小、数据传输效率高，适合对可靠性要求不高的应用程序，如果既要使用 UDP 高效的同时也想要保证数据的可靠性，那么就需要在应用程序中对收发的数据进行验证，这样的程序有很多，例如 DNS、TFTP、SNMP 等。<!--more-->

一个 UDP 数据报多大合适呢？简单的搜索了下，有文章说 UDP 最大载荷为 1472 字节，这里不敢确保这个值的正确性，只是简单的提及一下，大家好有个概念，实际项目里需要的时候各位在去看看相关的文档，这不是我们这里的重点。

UDP 不存在粘包问题，所以读取 UDP 数据报比较简单，但是 TCP 编程的时候我们必须自己解决粘包的问题。

**UDP 编程有三种模型用的比较多**

* [单播: Unicast](/qtbook-network-udp-unicast)
* [广播: Broadcast](/qtbook-network-udp-broadcast)
* [组播: Multicast](/qtbook-network-udp-multicast)

**为了能更形象的介绍 UDP 编程，假设我们的网络环境如下有 7 台电脑**

* 发送数据报的电脑：Source
* 接收数据报的电脑：
    * Destination 1
    * Destination 2
    * Destination 3
    * Destination 4
    * Destination 5
    * Destination 6

## 参考资料
[The Difference Between Unicast, Multicast and Broadcast Messages](http://www.utilizewindows.com/networking/basics/422-the-difference-between-unicast-multicast-and-broadcast-messages)