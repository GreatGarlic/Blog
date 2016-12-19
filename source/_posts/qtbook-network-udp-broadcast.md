---
title: 广播
date: 2016-12-19 18:46:00
tags: QtBook
---
“安红，额想你” 这句话曾经红的一塌糊涂，以至于人们只记住了这句话而忘了出自这句话的电影《有话好好说》：张艺谋骑着个破三轮出场，用陕西话喊：“安红，俺想你，想你想得想睡觉”：
![](/img/qtbook/network/Network-UDP-Broadcast-Movie-1.jpg)

注意，老谋子手里有个大喇叭，对着喇叭一喊，那就炸了锅了，周围的人都听到了。Broadcast 和用喇叭喊话很相似，消息一发送，就能被同一个局域网里的所有电脑收到。对着喇叭喊话对应于发送 Broadcast 消息，周围对应于局域网，声音的传递是有范围的，Broadcast 的消息也只能是局域网内被收到。<!--more-->

Broadcast 消息的目标地址使用的是广播地址，即 `255.255.255.255`，路由器不会把 Broadcast 的消息路由到外网。这是十分容易理解的，想像一下，如果路由器能把 Broadcast 信息广播到外网，我们只要买几台强大的机器，不停的向公共网路发送大量的 Broadcast 垃圾消息，每一条消息都会辐射出去，会发送到世界上的任何一台联网电脑，很容易就占满网络的带宽，导致整个互联网瘫痪，你创造了历史。这也是为什么 IP 协议的设计者故意没有定义互联网范围的广播机制。

假如我们小区所有人家的电脑都在一个局域网里，大家的关系也很好，经常一起组织活动，这次要通知大家明天晚上大家一起组织个篝火晚会，高科技年代，我们不用喇叭喊话了，而是用电脑发消息通知大家。如果使用 Unicast，首先需要知道每家每户的 IP 地址（很可能需要我们一家一家地去统计这些 IP 地址），然后给每个 IP 单独的发送消息。现在我们学会了 Broadcast，不需要知道每家得 IP 了，只需要使用广播地址 `255.255.255.255` 发送 Broadcast 消息，然后所有人家都收到明天晚上有个篝火晚会的消息了，是不是很方便？

下图是 Broadcast 的示意图，Source 发送 Broadcast 消息，所有的 Destination 都能收到这个消息。
![](/img/qtbook/network/Network-UDP-Broadcast.png)

## Broadcast 的消息发送程序

Broadcast 发送消息的程序和 Unicast 的几乎完全一样，只需要把  
`udpSocket->writeDatagram(data.toUtf8(), QHostAddress::LocalHost, 13930)`  
替换为  
`udpSocket->writeDatagram(data.toUtf8(), QHostAddress::Broadcast, 13930)`  
即把 Destination 的具体地址换为广播地址，这里我们只列出 main 函数，其他部分参考 Unicast 的程序。

```cpp
int main(int argc, char *argv[]) {
    QCoreApplication a(argc, argv);
    QUdpSocket *udpSocket = new QUdpSocket();
    int messageCount = 10;

    for (int i = 0; i < messageCount; ++i) {
        QString data = generateMessage(i, '-');
        // 使用 Broadcast 发送消息
        udpSocket->writeDatagram(data.toUtf8(), QHostAddress::Broadcast, 13930);
        qDebug() << data;
    }

    return a.exec();
}
```

## Unicast 的消息接收程序
Broadcast 的消息接收程序和 Unicast 的消息接收程序也是几乎完全一样，只需要把  
`udpSocket->bind(13930, QUdpSocket::DontShareAddress)`  
替换为  
`udpSocket->bind(13930, QUdpSocket::ShareAddress)`  
这里我们只列出 Broadcast 接收程序的构造函数，其他部分和 Unicast 的程序都一样。

```cpp
BroadcastReceiver::BroadcastReceiver(QObject *parent) : QObject(parent) {
    udpSocket = new QUdpSocket();
    udpSocket->bind(13930, QUdpSocket::ShareAddress);
    connect(udpSocket, SIGNAL(readyRead()), this, SLOT(readPendingDatagrams()));
}
```

