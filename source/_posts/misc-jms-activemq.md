---
title: JMS + ActiveMQ
date: 2016-11-20 09:47:54
tags: Misc
---
JMS 的全称是 Java Message Service，即 Java 消息服务，ActiveMQ 实现了 JMS 的接口。它主要用于在生产者和消费者之间进行消息传递，生产者负责产生消息，而消费者负责接收消息(生产者和消费者可以在同一个应用中，也可以不在同一个应用中)。应用到实际的业务需求中的话我们可以在特定的时候利用生产者生成一消息，并进行发送，对应的消费者在接收到对应的消息后去完成对应的业务逻辑。一个典型的应用例如用户注册后需要发送验证邮件，因为发送邮件是一个耗时任务，如果在注册的逻辑代码中发送邮件的话系统的响应就会很慢，可以在用户注册后立即返回，并把发送邮件的任务通过 JMS 发送到消息队列中，然后另一个专门负责发送邮件的服务从 MQ 里获取发送邮件的消息发送邮件。

> 如果是同一个程序里通讯的话，可以使用 Spring Event。

**消息有两种类型:**

* `点对点`: 一个消息只能被一个消费者接收处理
* `发布/订阅模式`: 一个消息能同时被多个消费者接收处理

**消息生产者使用步骤:**

* 配置 ConnectionFactory
* 配置 Destination，也就是队列
* 创建消息生产者对象
* 发送消息

**消息消费者使用步骤:**

* 配置 ConnectionFactory
* 创建消息消费者对象
* 配置消息监听容器
* 有消息到达时消息的消费者的 `onMessage()` 方法会被自动调用

> 程序运行前当然要先启动 ActiveMQ

<!--more-->

下面介绍 JMS 的使用，消息的生产者是一个 Web 应用，通过访问 URL 发送消息，消费者是一个普通的 Java 应用程序。

## Gradle 依赖
```groovy
compile(
        "org.springframework:spring-jms:4.3.0.RELEASE",
        "org.apache.activemq:activemq-core:5.7.0",
        "org.apache.activemq:activemq-pool:5.14.1"
)
```

## 消息生产者
### 项目目录
```
└── main
    ├── java
    │   └── com
    │       └── xtuer
    │           ├── controller
    │           │   └── ProducerController.java
    │           └── jms
    │               └── MessageProducer.java
    ├── resources
    │   └── config
    │       ├── jms.xml
    │       └── spring-mvc.xml
    └── webapp
        └── WEB-INF
            ├── view
            └── web.xml
```

### jms.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- 1. 真正可以创建 Connection 的 ConnectionFactory，由对应的 JMS 服务厂商提供-->
    <bean id="targetConnectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
        <property name="brokerURL" value="tcp://localhost:61616"/>
    </bean>
    <bean id="pooledConnectionFactory" class="org.apache.activemq.pool.PooledConnectionFactory">
        <property name="connectionFactory" ref="targetConnectionFactory"/>
        <property name="maxConnections" value="10"/>
    </bean>
    <bean id="connectionFactory" class="org.springframework.jms.connection.SingleConnectionFactory">
        <property name="targetConnectionFactory" ref="pooledConnectionFactory"/>
    </bean>

    <!-- 2.1. 消息目的地，点对点的 -->
    <bean id="queueDestination" class="org.apache.activemq.command.ActiveMQQueue">
        <constructor-arg value="testQueue"/>
    </bean>

    <!-- 2.2 消息目的地，一对多的 -->
    <bean id="topicDestination" class="org.apache.activemq.command.ActiveMQTopic">
        <constructor-arg value="testTopic"/>
    </bean>

    <!-- 3. Spring 提供的 JMS 工具类，用来消息发送 -->
    <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
        <property name="connectionFactory" ref="connectionFactory"/>
    </bean>

    <!-- 4. 消息的生产者 -->
    <bean id="messageProducer" class="com.xtuer.jms.MessageProducer">
        <property name="jmsTemplate" ref="jmsTemplate"/>
    </bean>
</beans>
```

### MessageProducer.java
> 使用 `jmsTemplate` 进行消息发送

```java
package com.xtuer.jms;

import org.springframework.jms.core.JmsTemplate;
import org.springframework.jms.core.MessageCreator;

import javax.jms.Destination;
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.Session;

public class MessageProducer {
    private JmsTemplate jmsTemplate;

    public void sendMessage(Destination destination, final String message) {
        System.out.println("---------------生产者发了一个消息：" + message);

        jmsTemplate.send(destination, new MessageCreator() {
            @Override
            public Message createMessage(Session session) throws JMSException {
                return session.createTextMessage(message);
            }
        });
    }

    public void sendMessage(final String message) {
        System.out.println("---------------生产者发了一个消息：" + message);

        // 消息的目的地使用字符串的队列名字表示, 而不是 Destination 对象, 这样就会省事一些
        jmsTemplate.send("default-destination", new MessageCreator() {
            @Override
            public Message createMessage(Session session) throws JMSException {
                // Message.setIntProperty("messageType", 1) 区别消息的类型
                return session.createTextMessage(message);
            }
        });
    }

    public void setJmsTemplate(JmsTemplate jmsTemplate) {
        this.jmsTemplate = jmsTemplate;
    }
}
```

### ProducerController.java
```java
package com.xtuer.controller;

import com.xtuer.jms.MessageProducer;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.annotation.Resource;
import javax.jms.Destination;

@Controller
public class ProducerController {
    @Resource(name = "queueDestination")
    private Destination queueDestination;

    @Resource(name = "topicDestination")
    private Destination topicDestination;

    @Resource(name = "messageProducer")
    private MessageProducer producer;

    @GetMapping("/test-queue")
    @ResponseBody
    public String testQueue() {
        String message = "Queue: " + System.nanoTime();
        producer.sendMessage(queueDestination, message);

        return message;
    }

    @GetMapping("/test-topic")
    @ResponseBody
    public String testTopic() {
        String message = "Topic: " + System.nanoTime();
        producer.sendMessage(topicDestination, message);

        return message;
    }
}
```

## 消息消费者
### 项目目录
```
└── main
    ├── java
    │   ├── Main.java
    │   └── com
    │       └── xtuer
    │           └── jms
    │               └── MessageConsumer.java
    └── resources
        └── jms.xml
```

### jms.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:jms="http://www.springframework.org/schema/jms"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/jms
       http://www.springframework.org/schema/jms/spring-jms.xsd">
    <!-- 1. 真正可以创建 Connection 的 ConnectionFactory，由对应的 JMS 服务厂商提供 -->
    <bean id="targetConnectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
        <property name="brokerURL" value="tcp://localhost:61616"/>
    </bean>
    <bean id="pooledConnectionFactory" class="org.apache.activemq.pool.PooledConnectionFactory">
        <property name="connectionFactory" ref="targetConnectionFactory"/>
        <property name="maxConnections" value="10"/>
    </bean>
    <bean id="connectionFactory" class="org.springframework.jms.connection.SingleConnectionFactory">
        <property name="targetConnectionFactory" ref="pooledConnectionFactory"/>
    </bean>

    <!-- 2. 消息的消费者 -->
    <bean id="messageConsumer" class="com.xtuer.jms.MessageConsumer"/>

    <!-- 3. 消息监听容器: 配置消息监听器监听的 destination -->
    <jms:listener-container connection-factory="connectionFactory" destination-type="queue">
        <jms:listener destination="testQueue" ref="messageConsumer"/>
    </jms:listener-container>
    <jms:listener-container connection-factory="connectionFactory" destination-type="topic">
        <jms:listener destination="testTopic" ref="messageConsumer"/>
    </jms:listener-container>
</beans>
```

> 一个消息的消费者可以同时接收多个队列里的消息。  
> 
> 上面使用了 `jms:listener-container` 定义消息监听容器，这种方式比较简洁，推荐使用:
> 
> * 目的地的名字直接使用字符串即可，不需要定义目的地的对象
> * 配置队列的类型: `destination-type`
> * 配置 acknowledge: `<jms:listener-container connection-factory="connectionFactory" acknowledge ="client">`
> 
> 当然也可以使用下面的方式定义消息监听容器:
> 
```xml
<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="messageListener" ref="messageConsumer"/>
    <property name="destination" ref="queueDestination"/> <!--队列对象-->
</bean>
或者
<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="messageListener" ref="messageConsumer"/>
    <property name="destinationName" value="testQueue"/> <!--队列名字-->
</bean>
```

### MessageConsumer.java
```java
package com.xtuer.jms;

import com.alibaba.fastjson.JSON;

import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageListener;
import javax.jms.TextMessage;

public class MessageConsumer implements MessageListener {
    public void onMessage(Message message) {
        // 这里我们知道生产者发送的就是一个纯文本消息，所以这里可以直接进行强制转换
        TextMessage textMsg = (TextMessage) message;

        try {
            // System.out.println(JSON.toJSONString(textMsg));
            System.out.println("接收到一个纯文本消息：" + textMsg.getText());
        } catch (JMSException e) {
            e.printStackTrace();
        }
    }
}
```

### Main.java
```java
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Main {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:jms.xml");
        context.start(); // 启动 Spring 容器，MessageConsumer 会自动接收到消息
    }
}
```

## 测试
* 访问 <http://localhost:8080/test-queue>
    * 消息生产者控制台输出: 生产者发了一个消息：Queue: 368904478401020
    * 消息消费者控制台输出: 接收到一个纯文本消息：Queue: 368904478401020
* 访问 <http://localhost:8080/test-topic>
    * 消息生产者控制台输出: 生产者发了一个消息：Topic: 365706083141038
    * 消息消费者控制台输出: 接收到一个纯文本消息：Topic: 365706083141038
