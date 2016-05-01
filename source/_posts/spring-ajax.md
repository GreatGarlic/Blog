---
title: SpringMVC 处理 Ajax 映射
date: 2016-04-17 21:51:01
tags: [Spring, Ajax]
---

SpringMVC 返回 Json 数据给前端是件很简单的事，但是 SpringMVC 的 Controller 接收前端传来的 Json 数据却不那么容易，前端和后端都要很小心，有一点不对就会出错。

<!--more-->

## 前端网页
* type 不能为 `GET`
* 要有 `contentType: 'application/json'`
* data 不能直接是 JS 对象，需要使用 `JSON.stringify(data)` 序列化一下

```js
Topic.prototype.update = function() {
    var data = {id: this.id, content: this.content, items: [1, 2, 3, 4]};
    
    $.ajax({
        url: urls.topic.replace('{id}', this.id),
        type: 'PUT', // 1. 不能是 GET
        dataType: 'json',
        contentType: 'application/json', // 2. 少了就会报错
        data: JSON.stringify(data) // 3. data 需要序列化一下
    })
    .done(function(result) {
        console.log(result);
        self.show();
    })
    .fail(function(error) {
        console.log(error.responseText);
    });
}
```

## 服务器端
使用 `@RequestBody` 接收前端 AJAX 传来的 Json 参数，不能使用 @RequestParam 等。

* 如果只想取某一个参数，可以使用 Map

    ```java
    @RequestMapping(value = "/topic/{id}", method = RequestMethod.PUT)
    @ResponseBody
    public String updateTopic(@RequestBody Map map) {
        System.out.println("-----------> " + map.get("id") + "" + map.get("content"));
    
        return "{successful: " + map.get("content") + "}";
    }
    ```

* 把 Json 数据映射为一个 Java Bean

    ```java
    @RequestMapping(value = "/topic/{id}", method = RequestMethod.PUT)
    @ResponseBody
    public String updateTopic(@RequestBody Topic topic) {
        System.out.println("-----------> " + topic.getContent() + ", " + topic.getItems());
    
        return "{successful: " + topic.getContent() + "}";
    }
    ```

    ```java
    public class Topic {
        private int id;
        private String content;
        private List<Integer> items;
        ...
    }
    ```

## 参考
* [SpringMVC @RequestBody接收Json对象字符串](http://www.cnblogs.com/quanyongan/archive/2013/04/16/3024741.html)

> 以前，一直以为在 SpringMVC 环境中，@RequestBody 接收的是一个 Json 对象，一直在调试代码都没有成功，后来发现，`其实 @RequestBody 接收的是一个 Json 对象的字符串，而不是一个 Json 对象`。然而在 ajax 请求往往传的都是 Json 对象，后来发现用 JSON.stringify(data) 的方式就能将对象变成字符串。同时 ajax 请求的时候也要指定 dataType: "json", contentType: "application/json" 这样就可以轻易的将一个对象或者 List 传到 Java 端，使用 @RequestBody 即可绑定对象或者 List。

