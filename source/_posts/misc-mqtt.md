---
title: ActiveMQ 的 MQTT
date: 2016-11-28 13:58:53
tags: [Misc, Util]
---

MQTT 有很多开源的实现，`org.fusesource.mqtt-client` 是其中之一，使用起来也比较简单，Github 地址为 [https://github.com/fusesource/mqtt-client](https://github.com/fusesource/mqtt-client)，而且其实现了连接断开后自动重连的机制

下面的程序使用 MQTT-Client 实现:

* Publisher
* Subscriber
* PublisherAndSubscriber
* FutureMqttPublisher

实际使用中，即要向 ActiveMQ 发送消息，同时也要从 ActiveMQ 订阅消息，所以参考下面的 Demo PublisherAndSubscriber 更为实用.

<!--more-->

## Gradle 依赖

```groovy
compile "org.fusesource.mqtt-client:mqtt-client:1.11"
```

## Mqtt 的 Publisher

MqttPublisher 只用于向 ActiveMQ 发送消息.

```java
package com.xtuer.mqtt;

import org.fusesource.hawtbuf.Buffer;
import org.fusesource.hawtbuf.UTF8Buffer;
import org.fusesource.mqtt.client.*;

import java.net.URISyntaxException;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

/**
 * Mqtt 的 publisher, 发送消息到 ActiveMQ
 */
public class MqttPublisher {
    private static final String TOPIC_NAME = "foo";
    private static final String WILL_TOPIC_NAME = "foo-will";
    private static final String HOST = "tcp://localhost:1883";
    private static boolean connectedToServer = false;
    private static int messageNumber = 0;

    public static void main(String[] args) throws URISyntaxException {
        final Promise<Buffer> result = new Promise<Buffer>();
        MQTT mqtt = new MQTT();
        mqtt.setHost(HOST);
        mqtt.setKeepAlive((short)3); // 默认是 30 秒
        mqtt.setReconnectDelay(500); // 500 毫秒重连一次, 默认是 10 毫秒
        mqtt.setReconnectDelayMax(500);
        mqtt.setWillTopic(WILL_TOPIC_NAME);
        mqtt.setWillQos(QoS.AT_LEAST_ONCE);
        mqtt.setWillMessage("Will message: My ID is A, I have left.");

        final CallbackConnection connection = mqtt.callbackConnection();

        // 给 connection 添加事件监听器
        // 1. 当有消息到达时会调用 onPublish() 方法
        // 2. 连上服务器时调用 onConnected() 方法
        // 3. 和服务器的连接断开时调用 onDisconnected() 方法
        connection.listener(new Listener() {
            /**
             * 成功连接上服务器
             */
            @Override
            public void onConnected() {
                connectedToServer = true;
                System.out.println("Connected to server");
            }

            /**
             * 从服务器断开连接
             */
            @Override
            public void onDisconnected() {
                connectedToServer = false;
                System.out.println("Disconnected from server");
            }

            @Override
            public void onFailure(Throwable value) {
                System.out.println("Failure: " + value);
                result.onFailure(value);
                connection.disconnect(null);
            }
        });

        // 连接服务器
        connection.connect(null); // Callback 为 null 会抛出一个异常, 不过不影响

        // 不停的发消息
        Executors.newScheduledThreadPool(1).scheduleAtFixedRate(new Runnable() {
            @Override
            public void run() {
                if (connectedToServer) {
                    String message = (++messageNumber) + " - A - 时间: " + System.currentTimeMillis();
                    connection.publish(TOPIC_NAME, message.getBytes(), QoS.AT_LEAST_ONCE, false, null);
                    System.out.println("Send: " + message);
                }
            }
        }, 100, 300, TimeUnit.MILLISECONDS);
    }
}
```

## Mqtt 的 Subscriber

MqttSubscriber 只用于从 ActiveMQ 订阅消息.

```java
package com.xtuer.mqtt;

import org.fusesource.hawtbuf.Buffer;
import org.fusesource.hawtbuf.UTF8Buffer;
import org.fusesource.mqtt.client.*;

import java.net.URISyntaxException;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

/**
 * Mqtt 的 subscriber, 订阅消息
 */
public class MqttSubscriber {
    private static final String TOPIC_NAME = "foo";
    private static final String WILL_TOPIC_NAME = "foo-will";
    private static final String HOST = "tcp://localhost:1883";

    public static void main(String[] args) throws URISyntaxException {
        final Promise<Buffer> result = new Promise<Buffer>();
        MQTT mqtt = new MQTT();
        mqtt.setHost(HOST);
        mqtt.setWillQos(QoS.AT_LEAST_ONCE);
        mqtt.setKeepAlive((short)3); // 默认是 30 秒
        mqtt.setReconnectDelay(500); // 500 毫秒重连一次, 默认是 10 毫秒
        mqtt.setReconnectDelayMax(500);

        final CallbackConnection connection = mqtt.callbackConnection();

        // 给 connection 添加事件监听器
        // 1. 当有消息到达时会调用 onPublish() 方法
        // 2. 连上服务器时调用 onConnected() 方法
        // 3. 和服务器的连接断开时调用 onDisconnected() 方法
        connection.listener(new Listener() {
            /**
             * 收到订阅的消息
             * @param topic 订阅队列的名字
             * @param payload 消息的内容
             * @param onComplete
             */
            @Override
            public void onPublish(UTF8Buffer topic, Buffer payload, Runnable onComplete) {
                result.onSuccess(payload);
                onComplete.run();

                // 消息处理逻辑
                System.out.println("Receive: " + new String(payload.toByteArray()));
            }

            /**
             * 成功连上服务器
             */
            @Override
            public void onConnected() {
                System.out.println("Connected to server");
            }

            /**
             * 从服务器断开连接
             */
            @Override
            public void onDisconnected() {
                System.out.println("Disconnected from server");
            }

            @Override
            public void onFailure(Throwable value) {
                System.out.println("Failure: " + value);
                result.onFailure(value);
                connection.disconnect(null);
            }
        });

        // 连接服务器，订阅 Topic，这里和 Publisher 有区别
        connection.connect(new Callback<Void>() {
            /**
             * 第一次连接上服务器后订阅 topic. 没有放到 listener 的 onConnected() 方法,
             * 是因为重连后 onConnected() 会被再次调用, 订阅会被执行多次, 而 onSuccess()
             * 只有第一次连接上时会被执行, 这样订阅就只执行了一次.
             *
             * @param v Unknown parameter
             */
            @Override
            public void onSuccess(Void v) {
                // 可以同时订阅几个 topic
                Topic[] topics = {new Topic(TOPIC_NAME, QoS.AT_LEAST_ONCE), new Topic(WILL_TOPIC_NAME, QoS.AT_LEAST_ONCE)};

                connection.subscribe(topics, new Callback<byte[]>() {
                    @Override
                    public void onSuccess(byte[] value) {
                    }

                    @Override
                    public void onFailure(Throwable value) {
                        result.onFailure(value);
                        connection.disconnect(null);
                    }
                });
            }

            @Override
            public void onFailure(Throwable value) {
                result.onFailure(value);
            }
        });

        // 不让程序结束
        Executors.newScheduledThreadPool(1).scheduleAtFixedRate(new Runnable() {
            @Override
            public void run() {}
        }, 1, 1, TimeUnit.SECONDS);
    }
}
```

## MqttPublisherAndSubscriber

MqttPublisherAndSubscriber 既是 Mqtt publisher 也是 subscriber.

```java
package com.xtuer.mqtt;

import org.fusesource.hawtbuf.Buffer;
import org.fusesource.hawtbuf.UTF8Buffer;
import org.fusesource.mqtt.client.*;

import java.net.URISyntaxException;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

/**
 * 同一个类既是 Mqtt publisher 也是 subscriber
 */
public class MqttPublisherAndSubscriber {
    private static final String TOPIC_NAME = "foo";
    private static final String WILL_TOPIC_NAME = "foo-will";
    private static final String HOST = "tcp://localhost:1883";
    private static boolean connectedToServer = false;
    private static int messageNumber = 0;

    public static void main(String[] args) throws URISyntaxException {
        final Promise<Buffer> result = new Promise<Buffer>();
        MQTT mqtt = new MQTT();
        mqtt.setHost(HOST);
        mqtt.setKeepAlive((short)3); // 默认是 30 秒
        mqtt.setReconnectDelay(500); // 500 毫秒重连一次, 默认是 10 毫秒
        mqtt.setReconnectDelayMax(500);
        mqtt.setWillTopic(WILL_TOPIC_NAME);
        mqtt.setWillQos(QoS.AT_LEAST_ONCE);
        mqtt.setWillMessage("Will message: My ID is 0000, I have left.");

        final CallbackConnection connection = mqtt.callbackConnection();
        // 给 connection 添加事件监听器
        // 1. 当有消息到达时会调用 onPublish() 方法
        // 2. 连上服务器时调用 onConnected() 方法
        // 3. 和服务器的连接断开时调用 onDisconnected() 方法
        connection.listener(new Listener() {
            /**
             * 收到订阅的消息
             * @param topic 订阅队列的名字
             * @param payload 消息的内容
             * @param onComplete
             */
            @Override
            public void onPublish(UTF8Buffer topic, Buffer payload, Runnable onComplete) {
                result.onSuccess(payload);
                onComplete.run();

                // 消息处理逻辑
                System.out.println("Receive: " + new String(payload.toByteArray()));
            }

            /**
             * 成功连接上服务器
             */
            @Override
            public void onConnected() {
                connectedToServer = true;
                System.out.println("Connected to server");
            }

            /**
             * 从服务器断开连接
             */
            @Override
            public void onDisconnected() {
                connectedToServer = false;
                System.out.println("Disconnected from server");
            }

            @Override
            public void onFailure(Throwable value) {
                System.out.println("Failure: " + value);
                result.onFailure(value);
                connection.disconnect(null);
            }
        });

        // 连接服务器
        connection.connect(new Callback<Void>() {
            /**
             * 第一次连接上服务器后订阅 topic. 没有放到 listener 的 onConnected() 方法,
             * 是因为重连后 onConnected() 会被再次调用, 订阅会被执行多次, 而 onSuccess()
             * 只有第一次连接上时会被执行, 这样订阅就只执行了一次.
             *
             * @param v Unknown parameter
             */
            @Override
            public void onSuccess(Void v) {
                // 可以同时订阅几个 topic
                Topic[] topics = {new Topic(TOPIC_NAME, QoS.AT_LEAST_ONCE), new Topic(WILL_TOPIC_NAME, QoS.AT_LEAST_ONCE)};

                connection.subscribe(topics, new Callback<byte[]>() {
                    @Override
                    public void onSuccess(byte[] value) {
                    }

                    @Override
                    public void onFailure(Throwable value) {
                        result.onFailure(value);
                        connection.disconnect(null);
                    }
                });
            }

            @Override
            public void onFailure(Throwable value) {
                result.onFailure(value);
            }
        });

        // 不停的发消息
        Executors.newScheduledThreadPool(1).scheduleAtFixedRate(new Runnable() {
            @Override
            public void run() {
                if (connectedToServer) {
                    String message = (++messageNumber) + " - B - 时间: " + System.currentTimeMillis();
                    connection.publish(TOPIC_NAME, message.getBytes(), QoS.AT_LEAST_ONCE, false, null);
                    System.out.println("Send: " + message);
                }
            }
        }, 100, 300, TimeUnit.MILLISECONDS);
    }
}
```

## 命令

* 启动 ActimeMQ: activemq start
* 关闭 ActimeMQ: activemq stop

## 测试一

1. 启动 ActiveMQ
2. 运行 MqttPublisherAndSubscriber

   ```
   Connected to server
   Send: 1 - B - 时间: 1457234699726
   Receive: 1 - B - 时间: 1457234699726
   Send: 2 - B - 时间: 1457234700026
   Receive: 2 - B - 时间: 1457234700026
   Send: 3 - B - 时间: 1457234700325
   ...
   ```

3. 关闭 ActiveMQ
4. 控制台输出 Disconnected from server 后一直等待

   ```
   ...
   Receive: 19 - B - 时间: 1457234796472
   Send: 20 - B - 时间: 1457234796745
   Receive: 20 - B - 时间: 1457234796745
   Disconnected from server
   ```

5. 启动 ActiveMQ
6. 过几秒，控制台输出 Connected to server，程序自动的重新连接上服务器了

   ```
   Disconnected from server
   Connected to server
   Send: 21 - B - 时间: 1457234862175
   Receive: 21 - B - 时间: 1457234862175
   Send: 22 - B - 时间: 1457234862477
   Receive: 22 - B - 时间: 1457234862477
   ```

## 测试二

1. 启动 ActiveMQ
2. 运行 MqttPublisherAndSubscriber（和 ActiveMQ 不在同一台机器上，注意修改程序里连接的地址）

   ```
   Connected to server
   Send: 1 - B - 时间: 1457234699726
   Receive: 1 - B - 时间: 1457234699726
   Send: 2 - B - 时间: 1457234700026
   Receive: 2 - B - 时间: 1457234700026
   Send: 3 - B - 时间: 1457234700325
   ...
   ```

3. 关闭无线网或者拔掉网线断开网络 (不是 关闭 ActiveMQ)
4. 订阅的数据是没了，但是还会不停的发送数据，并没有发现连接已经断开了
5. 过了很久，才会输出 Disconnected from server，也就是说，这种情况下不能及时的发现网络已经断开了


## 测试三

1. 启动 ActiveMQ
2. 运行 MqttPublisher 和 MqttSubscriber，数据的订阅和发布都是正常的
3. 关闭无线网或者拔掉网线断开网络
4. MqttSubscriber 很快就发现网络断开了，输出 Disconnected from server
5. MqttPublisher 和测试二一样，仍然不停的发送数据，过了很久才会发现网络断开了

> 推论: 在 callback 实现的代码里，如果只有 subscriber，不同什么情况下都能很快的发现网络断开的问题，但是只要有 publisher，断开网络的情况下需要很久才能发现网络异常

## 使用 Future 的方式实现 Publisher

Callback 方式实现的 Publisher 在特殊情况下不能及时的发现网络问题，使用 Future 实现的 Publisher 可以克服这个问题。

```java
package com.xtuer.mqtt.future;

import org.fusesource.mqtt.client.*;

import java.net.URISyntaxException;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class FutureMqttPublisher {
    private static final String TOPIC_NAME = "foo";
    private static final String WILL_TOPIC_NAME = "foo-will";
    private static final String HOST = "tcp://localhost:1883";
    private static int messageNumber = 0;

    public static void main(String[] args) throws URISyntaxException {
        MQTT mqtt = new MQTT();
        mqtt.setHost(HOST);
        mqtt.setKeepAlive((short)3); // 默认是 30 秒
        mqtt.setReconnectDelay(500); // 500 毫秒重连一次, 默认是 10 毫秒
        mqtt.setReconnectDelayMax(500);
        mqtt.setWillTopic(WILL_TOPIC_NAME);
        mqtt.setWillQos(QoS.AT_LEAST_ONCE);
        mqtt.setWillMessage("Will message: My ID is A, I have left.");

        final FutureConnection connection = mqtt.futureConnection();

        connection.connect().then(new Callback<Void>(){
            public void onSuccess(Void value) {
                System.out.println("Connected");
            }
            public void onFailure(Throwable e) {
                System.out.println("Failed");
            }
        });

        // 不停的发消息
        Executors.newScheduledThreadPool(1).scheduleAtFixedRate(new Runnable() {
            @Override
            public void run() {
                if (connection != null && connection.isConnected()) {
                    String message = (++messageNumber) + " - A - 时间: " + System.currentTimeMillis();
                    Future<Void> future = connection.publish(TOPIC_NAME, message.getBytes(), QoS.AT_LEAST_ONCE, false);

                    try {
                        future.await(2, TimeUnit.SECONDS); // 等待消息发送完成, 如果发送失败就会抛出异常, 例如网络断开的情况下
                        System.out.println("Send: " + message);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                } else {
                    System.out.println("Non-connected");
                }
            }
        }, 100, 300, TimeUnit.MILLISECONDS);
    }
}
```

## 测试四

和测试三的步骤一样，可以看到 Publisher 能及时的发现网络异常

## 问题和提示

**可以用同一个 MQTT 对象来创建 2 个不同的连接么？**

可以，例如用同一个 MQTT 对象来创建 CallbackConnection 用于接收消息，创建 FutureConnection 用于发送消息。

**KeepAlive 有什么作用?**

使用 mqtt.setKeepAlive((short)3) 设置连接的 KeepAlive 时间。每个连接都有自己的 KeepAlive，不同的连接的 KeepAlive 可以不同，如果服务器在连接的 KeepAlive 时间内没有收到消息，就会把这个连接断开。

**网络断开后重连会使用新的连接吗？**

在 KeepAlive 时间内仍然会重用原来的连接。超过 KeepAlive 则关闭原来的连接，重新创建一个。

网络断开（拔网线）后，客户端发现网络断开了，在 KeepAlive 时间内重新插上网线（服务器还没有把连接断开）并且客户端自动重连成功，会发现服务器端这个客户端的 Socket 被重用（IP 和 端口没变），而不是重新创建一个新的 Socket 连接，如果时间过长（大于 KeepAlive），那么就会重新创建一个连接，原来的连接被关掉。

> Socket 连接创建是需要三次握手的

**Socket 编程中怎么及时发现 Socket 断开?**

Socket 编程里，Socket.sendUrgentData() 成功就表示连接是好的，否则远程的连接就断开了，用来判断连接的有效性效果不错

> public void sendUrgentData(int data) throws IOException
>
> Send one byte of urgent data on the socket. The byte to be sent is the lowest eight bits of the data parameter(参数的第八位). The urgent byte is sent after any preceding writes to the socket OutputStream and before any future writes to the OutputStream.
>
> Parameters:
>
> data - The byte of data to send
>
> Throws:
>
> IOException - if there is an error sending the data.