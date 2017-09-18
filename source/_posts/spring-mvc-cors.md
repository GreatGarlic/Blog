---
title: Spring 中配置 CORS
date: 2016-06-10 21:21:07
tags: SpringMVC
---

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

## 在响应的 Http Header 中写入 CORS 信息
```java
    public String writeHeader(HttpServletResponse response) {
        response.setHeader("Access-Control-Allow-Origin", "*");
        response.setHeader("Access-Control-Allow-Methods", "GET, POST");
        ...
    }
```

## 在 Controller 上使用 `@CrossOrigin` 注解
```java
@CrossOrigin(origins = "*", maxAge = 3600)
@Controller
public class FooController {
    ...
}
```

## 在方法上使用 `@CrossOrigin` 注解
```java
    @CrossOrigin("http://domain2.com")
    @RequestMapping("/foo/{id}")
    public Account retrieve(@PathVariable Long id) {
        ...
    }
```

## Spring 的 xml 文件中配置全局的 `<mvc:cors>`
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

## 参考文档
* [AJAX 跨域访问](http://qtdebug.com/spring-web/AJAX%20跨域访问.html)
* [SpringMvc 解决跨域问题](http://my.oschina.net/wangnian/blog/689020?fromerr=v0Kie8WA)
