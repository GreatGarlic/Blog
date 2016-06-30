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
4. 查看 ActiveMQ 的信息，如有多少个 queues, topics, connections 等:
    
    ```
    URL: http://localhost:8161/admin
    Username: admin
    Password: admin
    ```

    ![](/img/misc/ActiveMQ-Admin.png)
5. 查看有多少个 Producer

    ```
    activemq-admin query TotalConnectionsCount=* | grep TotalProducerCount
    ```
5. 查看有多少个 Consumer

    ```
    activemq-admin query TotalConnectionsCount=* | grep TotalConsumerCount
    ```
5. 查看连接到 Broker 的连接的信息

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
6. 查看 Queue 和 Topic 的信息

    ```
    activemq-admin query -QTopic=*
    activemq-admin query -QQueue=*
    ```
7. Broker status

    > activemq-admin query --objname  
    > type=Broker,  
    > brokerName=localhost  
8. Connection Stats

    > activemq-admin query --objname  
    > type=Broker,   
    > brokerName=localhost,  
    > connector=clientConnectors,  
    > connectorName=<transport connector name>,  
    > connectionViewType=clientId,  
    > connectionName=* 
