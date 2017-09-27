---
title: ActiveMQ
date: 2016-06-29 16:34:36
tags: Misc
---

ActiveMQ 的常用命令和功能。

---

<!--more-->

1. 安装 ActiveMQ: `brew install activemq`
2. 启动 ActiveMQ: `activemq start`
3. 关闭 ActiveMQ: `activemq stop`
4. 查看当前连接数: 

    ```
    activemq-admin query --view CurrentConnectionsCount | grep CurrentConnectionsCount

    也可以使用 netstat: netstat -an | grep 61616 | wc -l
    ```
5. 网页中查看 ActiveMQ 的信息，如有多少个 queues, topics, connections 等:

    ```
    URL: http://localhost:8161/admin
    Username: admin
    Password: admin
    ```

    ![](/img/misc/ActiveMQ-Admin.png)
6. 查看有多少个 Producer

    ```
    activemq-admin query TotalConnectionsCount=* | grep TotalProducerCount
    ```
7. 查看有多少个 Consumer

    ```
    activemq-admin query TotalConnectionsCount=* | grep TotalConsumerCount
    ```
8. 查看连接到 Broker 的连接的信息

    ```
    activemq-admin query connectionName=*

    查看连接
    activemq-admin query connectionName=* | grep RemoteAddress | sort

    输出
    RemoteAddress = tcp://127.0.0.1:59420
    RemoteAddress = tcp://127.0.0.1:59420

    查看连接，去掉重复的行, -c 显示重复行的数字
    activemq-admin query connectionName=* | grep RemoteAddress | sort | uniq
    activemq-admin query connectionName=* | grep RemoteAddress | sort | uniq -c

    查看有多少个连接
    activemq-admin query connectionName=* | grep RemoteAddress | wc -l
    activemq-admin query connectionName=* | grep RemoteAddress | uniq | wc -l
    ```
9. 查看 Queue 和 Topic 的信息

    ```
    activemq-admin query -QTopic=*
    activemq-admin query -QQueue=*
    ```
10. Broker status

    > activemq-admin query \--objname  
    > type=Broker,  
    > brokerName=localhost  
11. Connection Stats

     > activemq-admin query \--objname  
     > type=Broker,   
     > brokerName=localhost,  
     > connector=clientConnectors,  
     > connectorName=<transport connector name>,  
     > connectionViewType=clientId,  
     > connectionName=* 

## 协议

修改 `conf/activemq.xml` 的 **transportConnectors**

```xml
<!--
    The transport connectors expose ActiveMQ over a given protocol to
    clients and other brokers. For more information, see:

    http://activemq.apache.org/configuring-transports.html
-->
<transportConnectors>
    <!-- DOS protection, limit concurrent connections to 1000 and frame size to 100MB -->
    <transportConnector name="openwire" uri="tcp://0.0.0.0:61616?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
    <transportConnector name="amqp" uri="amqp://0.0.0.0:5672?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
    <transportConnector name="stomp" uri="stomp://0.0.0.0:61613?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
    <transportConnector name="mqtt" uri="mqtt://0.0.0.0:1883?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
    <transportConnector name="ws" uri="ws://0.0.0.0:61614?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
</transportConnectors>
```

