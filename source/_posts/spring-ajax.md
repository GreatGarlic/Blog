---
title: SpringMVC 处理 Ajax 映射
date: 2016-04-17 21:51:01
tags: [Spring, Ajax]
---

SpringMVC 返回 Json 数据给前端是件很简单的事，但是 SpringMVC 的 Controller 接收前端 Ajax传来的 Json 数据却不那么容易，前端和后端都要很小心，有一点不对就会出错，需要分为 2 种情况处理:

* GET
* PUT, POST, DELETE

<!--more-->

## GET
一、浏览器端 Ajax 请求

* type 为 `GET`
* contentType 为 `application/json`
* data 是 JSON 对象，不能使用 `JSON.stringify(data)` 序列化

```js
<script>
$(document).ready(function() {
    $.ajax({
        url:  'ajax-test',
        type: 'GET',
        dataType: 'json',
        contentType: 'application/json',
        data: {age: 10} //JSON.stringify({age: 10})
    })
    .done(function(result) {
        console.log("success");
        console.log(result);
    })
    .fail(function(error) {
        console.log("error");
        console.log(error);
    });
});
</script>
```

二、服务器端

* method 为 `RequestMethod.GET`
* 使用 `@RequestParam` 接收参数

```java
@RequestMapping(value="ajax-test", method= RequestMethod.GET)
@ResponseBody
public String ajaxTest(@RequestParam int age) {
    System.out.println("==> age: " + age);
    return "{\"success\": true}";
}
```

> 服务器返回的数据需要注意，例如 `return "{success: true}"` 或者 `return success`，返回的不是一个 JSON 对象的格式，浏览器端会调用 `fail()`，不过这时看 error 的 `status` 却是 200。正确的 JSON 格式为 `"{\"success\": true}"`，属性名需要用双引号括起来。


## PUT, POST, DELETE
一、浏览器端 AJAX 请求

* type 为 `PUT`, `POST` 或 `DELETE`
* contentType 为 `application/json`
* data 不是 JSON 对象，需要使用 JSON.stringify(data) 序列化一下

```js
<script>
$(document).ready(function() {
    $.ajax({
        url:  'ajax-test',
        type: 'POST',
        dataType: 'json',
        contentType: 'application/json',
        data: JSON.stringify({age: 10})
    })
    .done(function(result) {
        console.log("success");
        console.log(result);
    })
    .fail(function(error) {
        console.log("error");
        console.log(error);
    });
});
</script>
```

二、服务器端

* method 为 `RequestMethod.PUT (POST, DELETE)`
* 使用 `@RequestBody ` 接收参数
    * 如果只想取某一个参数，可以使用 Map: `@RequestBody Map map`
    * 把 Json 数据映射为一个 Java Bean: `@RequestBody Topic topic`

```java
@RequestMapping(value="ajax-test", method= RequestMethod.POST)
@ResponseBody
public String ajaxGet(@RequestBody Map map) {
    System.out.println("==> age: " + map);
    return "{\"success\": true}";
}
```

## 参考
* [SpringMVC @RequestBody接收Json对象字符串](http://www.cnblogs.com/quanyongan/archive/2013/04/16/3024741.html)

> 以前，一直以为在 SpringMVC 环境中，@RequestBody 接收的是一个 Json 对象，一直在调试代码都没有成功，后来发现，`其实 @RequestBody 接收的是一个 Json 对象的字符串，而不是一个 Json 对象`。然而在 ajax 请求往往传的都是 Json 对象，后来发现用 JSON.stringify(data) 的方式就能将对象变成字符串。同时 ajax 请求的时候也要指定 dataType: "json", contentType: "application/json" 这样就可以轻易的将一个对象或者 List 传到 Java 端，使用 @RequestBody 即可绑定对象或者 List。

