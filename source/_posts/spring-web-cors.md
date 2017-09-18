---
title: Ajax 跨域访问
date: 2017-03-31 19:29:42
tags: SpringWeb
---
## 什么情况下是跨域访问？

URL: `prototype`://`hostname`:`port`/path，当 prototype，hostname(包括子域名)，port 中有任意一个和发起 request 的 URL 中对应部分不同时，就是跨域访问。例如从 <http://localhost/foo.html> 中发起到 <http://127.0.0.1/bar.php> 的 AJAX 请求就是跨域请求，这也是本地测试跨域请求的好办法。<!--more-->

## 使用 JSONP 实现 AJAX 跨域请求的例子

* 只支持 GET，不支持 POST
* Server 端返回的不是 JSON 数据，对返回的数据格式有特殊要求，为 `'callback(json)'` 格式的字符串

**后端 PHP 代码:**

```php
<?php
$jsonp = $_GET["jsonpCallback"];
echo $jsonp . '({name: "Bill", age: "100"})';
?>
```

**前端代码:**

```js
<!DOCTYPE html>
<html>

<body>
    <button id="button">MyButton</button>
</body>

<script type="text/javascript" src="jquery.js"></script>
<script type="text/javascript">
$(document).ready(function() {
    // 点击 MyButton 发起 JSONP 请求
    $("#button").click(function() {
        $.ajax({
            url: "http://127.0.0.1:8000/foo.php",
            type: "GET",
            dataType: "jsonp", // 表示要用 JSONP 进行跨域访问
            jsonp: "jsonpCallback", // 传递给服务器端请求处理程序或页面的，
                                    // 用于取得浏览器端 jsonp 回调函数名的函数名
                                    // (省略则用默认的，为 "callback")
            success: function(data) { // 随机生成回调函数名
                console.log("Name: " + data.name + ", Age: " + data.age);
            }
        });
    });
});
</script>
</html>
```

如果上面不用 success，而是用 jsonpCallback，则可使用字符串设置回调函数的名字，下面的代码功能和上面的一样

```js
<!DOCTYPE html>
<html>

<body>
    <button id="button">MyButton</button>
</body>

<script type="text/javascript" src="jquery.js"></script>
<script type="text/javascript">

function responseSuccess(data) {
    console.log("Name: " + data.name + ", Age: " + data.age);
}

$(document).ready(function() {
    // 点击 MyButton 发起 JSONP 请求
    $("#button").click(function() {
        $.ajax({
            url: "http://127.0.0.1:8000/foo.php",
            type: "GET",
            dataType: "jsonp", // 表示要用 JSONP 进行跨域访问
            jsonp: "jsonpCallback",
            jsonpCallback: "responseSuccess" // 默认为随机生成
        });
    });
});
</script>
</html>
```

**测试:**

1. 浏览器里输入 <http://localhost:8000/jsonp.html>
2. 点击按钮，控制台输出: `Name: Bill, Age: 100`
3. 用调试工具可以看到，用 success 的方式响应为  
   `jQuery211011382594271428881_1432652219543({name: "Bill", age: "100"})`  
   用 jsonpCallback 的方式响应为  
   `responseSuccess({name: "Bill", age: "100"})`

## 使用 HTML5 实现 AJAX 跨域访问

HTML5新的标准中，增加了 `Cross-Origin Resource Sharing` 特性，这个特性的出现使得跨域通信只需通过配置 HTTP 协议头来即可解决

* 支持 GET 和 POST
* Server 端返回的是 JSON 数据
* 在服务器端返回头加上访问控制 `Access-Control-Allow-Origin` 和 `Access-Control-Allow-Methods`

**后端 PHP 代码:**

```php
<?php
header("Access-Control-Allow-Origin: *");
header("Access-Control-Allow-Methods: GET, POST");

echo '{"name": "Bill", "age": "100"}';
?>
```

**前端是普通的 AJAX 访问的代码:**

```html
<!DOCTYPE html>
<html>

<body>
    <button id="button">MyButton</button>
</body>

<script type="text/javascript" src="jquery.js"></script>
<script type="text/javascript">
$(document).ready(function() {
    $("#button").click(function() {
        $.ajax({
            url: "http://127.0.0.1:8000/foo.php",
            type: "GET",
            dataType: "json",
            // jsonp: "jsonpCallback",
            success: function(data) {
                console.log("Name: " + data.name + ", Age: " + data.age);
            }
        });

        $.getJSON("http://127.0.0.1:8000/foo.php", function(data) {
            console.log("Name: " + data.name + ", Age: " + data.age);
        });

        $.get( "http://127.0.0.1:8000/foo.php", function(data) {
            console.log("Name: " + data.name + ", Age: " + data.age);
        }, "json");
    });
});
</script>
</html>
```

**测试:**

1. 浏览器里输入 <http://localhost:8000/json.html>
2. 点击按钮，控制台输出: <br>`Name: Bill, Age: 100`<br>`Name: Bill, Age: 100`<br>`Name: Bill, Age: 100`

## Spring 中配置 CORS 实现跨域访问

Ajax 以前要实现跨域访问，可以通过 JSONP、Flash 或者服务器中转的方式来实现，现在可以使用 `CORS`。

> 跨域资源共享（CORS ）是一种网络浏览器的技术规范，它为 Web 服务器定义了一种方式，允许网页从不同的域访问其资源，而这种访问是被同源策略所禁止的。CORS 系统定义了一种浏览器和服务器交互的方式来确定是否允许跨域请求。 它是一个妥协，有更大的灵活性，但比起简单地允许所有这些的要求来说更加安全。

<!--more-->

CORS 与 JSONP 相比，更为先进、方便和可靠:

1. JSONP只能实现GET请求，而CORS支持所有类型的HTTP请求。
2. 使用CORS，开发者可以使用普通的XMLHttpRequest发起请求和获得数据，比起JSONP有更好的错误处理。
3. JSONP主要被老的浏览器支持，它们往往不支持CORS，而绝大多数现代浏览器都已经支持了CORS

Spring 中使用 CORS 有几种方式:

* 在响应的 Http Header 中写入 CORS 信息，如 `Access-Control-Allow-Origin`
* 在 Controller 上使用 `@CrossOrigin` 注解
* 在方法上使用 `@CrossOrigin` 注解
* Spring 的 xml 文件中配置全局的 `<mvc:cors>`

### 在响应的 Http Header 中写入 CORS 信息

```java
    public String writeHeader(HttpServletResponse response) {
        response.setHeader("Access-Control-Allow-Origin", "*");
        response.setHeader("Access-Control-Allow-Methods", "GET, POST");
        ...
    }
```

### 在 Controller 上使用 `@CrossOrigin` 注解

```java
@CrossOrigin(origins = "*", maxAge = 3600)
@Controller
public class FooController {
    ...
}
```

### 在方法上使用 `@CrossOrigin` 注解

```java
    @CrossOrigin("http://domain2.com")
    @RequestMapping("/foo/{id}")
    public Account retrieve(@PathVariable Long id) {
        ...
    }
```

### Spring 的 xml 文件中配置全局的 `<mvc:cors>`

```xml
<mvc:cors>
    <mvc:mapping path="/**" />
</mvc:cors>
```

```xml
<mvc:cors>
    <mvc:mapping path="/api/**"
        allowed-origins="http://domain1.com, http://domain2.com"
        allowed-methods="GET, PUT"
        allowed-headers="header1, header2, header3"
        exposed-headers="header1, header2" 
        allow-credentials="false"
        max-age="3600" />

    <mvc:mapping path="/resources/**" allowed-origins="http://domain1.com" />
</mvc:cors>
```

> 推荐使用配置的方式。