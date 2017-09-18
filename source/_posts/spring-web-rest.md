---
title: 使用 REST
date: 2016-10-15 14:54:47
tags: SpringWeb
---
## 在 `web.xml` 里加上 HiddenHttpMethodFilter
```xml
<!-- 浏览器的 form 不支持 put, delete 等 method, 由该 filter 将 /blog?_method=delete 转换为标准的 http delete 方法 -->
<filter>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <servlet-name>springmvc</servlet-name>
</filter-mapping>
```

<!--more-->

## 新建 RestController
```java
package com.xtuer.controller;

import com.xtuer.bean.Result;
import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.*;
import java.util.Map;

@Controller
public class RestController {
    @GetMapping("/rest-form")
    public String restForm() {
        return "rest-form.fm";
    }

    @GetMapping("/rest/{id}")
    @ResponseBody
    public Result handleGet(@PathVariable int id, @RequestParam String name, ModelMap map) {
        map.addAttribute("id", id);
        map.addAttribute("name", name);
        return new Result(true, "GET handled", map);
    }

    // 查询
    @GetMapping("/rest")
    @ResponseBody
    public Result handleGet(@RequestParam String name) {
        return new Result(true, "GET handled", name);
    }

    // 更新
    @PutMapping("/rest")
    @ResponseBody
    public Result handlePut(@RequestBody Map map) {
        return new Result(true, "UPDATE handled", map);
    }

    // 创建
    @PostMapping("/rest")
    @ResponseBody
    public Result handlePost() {
        return new Result(true, "CREATE handled");
    }

    // 删除
    @DeleteMapping("/rest")
    @ResponseBody
    public Result handleDelete() {
        return new Result(true, "DELETE handled");
    }
}
```

## 新建一个网页 rest-form.fm
##### 页面里有 4 个按钮，分别为 `GET`, `PUT`, `POST`, `DELETE`
* `GET` 按钮发送 `GET` 请求
* `PUT` 按钮发送 `PUT` 请求
* `POST` 按钮发送 `POST` 请求
* `DELETE` 按钮发送 `DELETE` 请求

Form 表单默认支持 GET 和 POST 请求，但不支持 PUT 和 DELETE 请求。为了发送 PUT, DELETE请求，在 form 里添加一个隐藏域 _method 表明发送 PUT, DELETE 请求，使用 POST 提交。

```html
<!-- UUID: 982401A7-85F5-48AA-A2E6-D88A0960758D -->
<html>
<head>
    <meta http-equiv="content-type" content="text/html; charset=UTF-8"/>
    <style>
        body { padding: 25px 50px; }
        button { width: 100px; }
    </style>
</head>
<body>
    <!-- 测试 form 的不同 method，default HTML form 不支持 put and delete -->

    <form action="/rest" method="get">
        <input type="hidden" name="name"/>
        <button type="submit">Get</button>
    </form>

    <form action="/rest" method="post">
        <button type="submit">Post</button>
    </form>

    <form action="/rest" method="post">
        <input type="hidden" name="_method" value="put"/>
        <button type="submit">Put</button>
    </form>

    <form action="/rest" method="post">
        <input type="hidden" name="_method" value="delete"/>
        <button type="submit">Delete</button>
    </form>
</body>
</html>
```

## 测试
访问 <http://localhost:8080/rest-form> 查看输出

## 使用 AJAX 发送 REST 请求
Form 表单不支持提交 PUT，DELETE 请求，所以我们通过隐藏域的方式间接的达到了目的。但是 `AJAX 原生的就支持 GET, PUT, POST, DELETE 请求`。

请参考 [jQuery 的 REST 插件](/fe-rest)
