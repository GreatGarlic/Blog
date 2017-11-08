---
title: HTML5 使用 MQTT
date: 2017-09-27 16:54:14
tags: FE
---

HTML5 中也能使用 MQTT：

1. ActiveMQ 启用 MQTT，可参考 http://qtdebug.com/misc-activemq/

2. 启动 ActiveMQ: `activemq start`

3. 使用 MQTT 的 HTML 如下:

   ```html
   <html>

   <head>
       <title>Test Ws mqtt.js</title>
   </head>

   <body>
       <script src="./browserMqtt.js"></script>
       <script>
           // 虽然使用的是 MQTT，但底层还是使用 WebSocket 实现的，所以这里的端口需要使用 ActiveMQ 里 WS 的端口 61614，而不是 MQTT 的端口 1883
           var client = mqtt.connect('ws://127.0.0.1:61614'); // you add a ws:// url here
           client.subscribe('foo'); // 订阅 Topic

           client.on('message', function(topic, payload) {
               console.log([payload].join('')); // 提取消息需要使用 [].join()
           })

           client.publish('foo', 'Hello World!'); // 发送消息

           // 不停的发送消息进行测试
           setInterval(function() {
               client.publish('foo',  'Time: ' + new Date().getTime());
           }, 1000);
       </script>
   </body>

   </html>
   ```

4. 写一个 Java 的 MQTT 发布和订阅的程序一起测试，可参考 http://qtdebug.com/misc-mqtt/

下载 [browserMqtt.js](/download/browserMqtt.js.zip)，也可以[自己编译](https://github.com/mqttjs/MQTT.js)(新版本好像有问题，只能发消息，不能订阅消息)，详细文档请参考 https://github.com/mqttjs/MQTT.js