---
title: SpringMvc 响应 JSONP
date: 2017-04-20 14:12:30
tags: Spring-Web
---

SpringMvc 中处理 JSONP 需要注意响应的 Content-Type，如果为 text/plain 时在某些浏览器下就不能正确的执行 JSONP 的回调函数，认为其是不可执行的格式。

> SpringMvc 返回对象时会把其自动的转换为 JSON 字符串，并且设置 Content-Type 为 **application/json**，如果返回 String 的话则 Content-Type 会设置为 **text/plain**，使用 JSONP 时需要返回 JSONP 格式的 String，这时例如在高版本的 Chrome 中就会出错，因为 Content-Type 是 **text/plain**，而不是 **application/javascript**。<!--more-->

## 服务器端:

* 使用 produces 设置响应的 Content-Type
* 返回 JSONP 格式的字符串，例如 `jQuery21107523536464367151_1492668511201("Goo")`

```java
import com.alibaba.fastjson.JSONPObject;

@GetMapping(value="/jsonp-test", produces="application/javascript;charset=UTF-8")
@ResponseBody
public String jsonpTest(@RequestParam String callback) {
    JSONPObject jsonp = new JSONPObject(callback);
    jsonp.addParameter("Goo"); // 或则 Object

    return jsonp.toString();
}
```

## 浏览器端:

```js
$.ajax({
    url     : 'http://127.0.0.1:8080/jsonp-test',
    type    : 'GET',
    dataType: 'jsonp', // 表示要用 JSONP 进行跨域访问
    success : function(data) {
        console.log(data);
    }
});
```

