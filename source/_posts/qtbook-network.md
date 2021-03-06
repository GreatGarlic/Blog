---
title: 网络编程
date: 2016-12-16 11:37:54
tags: QtBook
---
**网络通讯无处不在**

* 用 QQ 和朋友聊天，发文字，发图片，发文件，语音聊天，视频聊天，然而，对方的 QQ 是怎么接受到我们发送的数据呢？
* 用浏览器上网，网页上的图片，JavaScript 文件，CSS 等是怎么下载来的呢？
* 使用 FTP 工具下载服务器上的文件
* 用 BT 软件下载电影
* 在线看电影，听歌
* 玩网络游戏
* 软件在线升级
* ...

用到网络通讯的地方太多太多，举不胜举，如果没有网络，电脑不能上网，手机不能上网，不能玩网络游戏，不能淘宝，不能在 12306 买火车票，不能百度，不能在线学习，不能用百度地图，不能团购，不能……，这样的世界将会是多么的无趣！

网络编程大家应该都听过 UDP，TCP，也许你还知道如 FileZilla 是用 FTP 传输协议来上传下载文件的，浏览器使用 HTTP 协议和服务器交互，BT 软件使用 P2P 协议下载文件，QQ 有自己的通讯协议，不同的软件很有可能定制了自己的传输协议，从网络上搜素，可以看到各种各样的协议，还有程序之间使用 MQ 通讯，网络通讯的库 Netty，Mina，高性能 TCP/UDP Socket 组件 HP-Socket，也许看到别人介绍网络编程时可能用到 FlatBuffers 等。

看的头都大了，我就是想写个小程序，在 2 台电脑之间发个文字消息，难道要了解上面这些各种各样的东西？吓得赶快去写个单机程序压压惊！

其实上面说的形形色色的东西，他们底层数据的传输都只是用了 2 个东西：UDP 和 TCP，完成上面发消息的这个小程序，只要了解基础的 UDP，TCP 编程就够了。所谓的通讯协议如 HTTP，FTP，P2P 等没有什么神秘的，他们定义的只不过是一个数据结构，某几个字节表示的是什么数据，有的数据项多些，有的数据项少些而已，数据按照通讯协议组织后使用 UDP 或者 TCP 来传输，对方接收到数据后，根据协议定义的格式解析数据，协议就是网络通讯中用的语言，只有语言通了，才能交流，否则就是鸡同鸭讲，不知所谓。

网络编程我们会介绍 UDP，TCP 基础编程，自定义通讯协议，使用 FlatBuffers 简化数据的序列化和反序列化，还会介绍使用 MQ 在不同的语言编写的程序之间进行网络通讯。
