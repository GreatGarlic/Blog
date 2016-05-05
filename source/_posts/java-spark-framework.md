---
title: 微 Web 服务的 REST 框架 Spark Framework
date: 2016-05-05 10:29:49
tags: Java
---

需要点击网页上的一个按钮打开本机上的文件或者修改本地的配置文件，由于安全的限制浏览器不能直接访问本地文件系统。为了实现这个功能，可以本地启动一个 Web 服务，浏览器访问这个 Web 服务(localhost)，然后使用 Web 服务中的代码来访问本地文件系统。为了启动一个 Web 服务，一般我们会选择 Tomcat，Jetty，Websphere 等作为 Web 服务器响应 HTTP 请求，对于我们这么小的一个需求，就有些重量级了，如果能像打开一个普通程序一样打开程序就能启动 Web 服务就好了。

<!--more-->

使用 [Spark Framework](http://sparkjava.com)，几行代码，就能启动一个嵌入式的 Web 服务器，默认端口是 `4567`，还不需要任何配置，是一个很好的选择

> Spark Framework is a simple and lightweight Java web framework built for rapid development. Spark's intention isn't to compete with Sinatra, or the dozen of similar web frameworks in different languages, but to provide a pure Java alternative for developers that want to (or are required to), develop their web application in Java. Spark is built around `Java 8's` lambda philosophy, which makes a typical Spark application a lot less verbose than most application written in other Java web frameworks.

> Spark focuses on being as simple and straight-forward as possible, without the need for cumbersome (XML) configuration, to enable very fast web application development in pure Java with minimal effort. It’s a totally different paradigm when compared to the overuse of annotations for accomplishing pretty trivial stuff seen in other web frameworks.

## Gradle 依赖
```groovy
'com.sparkjava:spark-core:2.5'
```

## Web 服务器程序
```java
import static spark.Spark.*;

public class HelloSpark {
    public static void main(String[] args) {
        get("/hello", (req, res) -> "Hello World");
    }
}
```

## 测试一
1. 运行 `HelloSpark`
2. 访问 <http://localhost:4567/hello>

## 访问本地文件
上面的程序启动了本地的 Web 服务，那么接下来就模拟点击网页上的按钮，然后打开本地文件吧

```java
import java.awt.*;
import java.io.File;

import static spark.Spark.get;

public class HelloSpark {
    public static void main(String[] args) {
        get("/open-file", (request, response) -> {
            Desktop desktop = Desktop.getDesktop();
            desktop.open(new File("/Users/Biao/Downloads/华夏对接.docx")); // 打开本地文件

            // 允许跨域访问
            response.header("Access-Control-Allow-Origin", "*");
            response.header("Access-Control-Allow-Methods", "GET, POST");
            response.type("application/json"); // 返回 Json 数据, 默认返回 html

            return "{\"success\": true, \"message\": \"\"}";
        });
    }
}
```

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <script src="http://libs.baidu.com/jquery/1.11.1/jquery.min.js"></script>
    <script>
    $(document).ready(function() {
        $('#button').click(access);
    });

    function access() {
        $.ajax({
            url: 'http://localhost:4567/hello',
            type: 'GET',
            dataType: 'json',
            statusCode: {
                404: function() { // 找不到指定的 URL
                    console.log('404: Page not found');
                },
                0: function() { // 连不上服务器
                    console.log('0: Server is not started');
                }
            }
        })
        .done(function(data) {
            console.log(data);
        })
        .fail(function(response) {
            console.log(response);
        })
        .always(function() {
            console.log("complete");
        });
    }
    </script>
</head>

<body>
    <button id="button">Press me</button>
</body>
</html>
```

## 测试二
1. 运行 `HelloSpark`
2. 访问 <http://localhost:4567/open-file>
3. 本地的文件被打开了
