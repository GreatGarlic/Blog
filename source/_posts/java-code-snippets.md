---
title: Java Code Snippets
date: 2016-06-02 13:22:45
tags: [Util, Java]
---

一些 Java 里常用的代码片段，例如 Bean to String 等。

<!--more-->

## Bean to String
使用 Apache Commons Lang3

```java
import org.apache.commons.lang3.builder.ReflectionToStringBuilder;
import org.apache.commons.lang3.builder.ToStringStyle;
import org.apache.commons.lang3.builder.ToStringBuilder;

System.out.println(ReflectionToStringBuilder.toString(obj));
System.out.println(ReflectionToStringBuilder.toString(obj, ToStringStyle.MULTI_LINE_STYLE));
System.out.println(ToStringBuilder.reflectionToString(p));
```

使用 Jackson，Json 的格式更漂亮

```java
import com.fasterxml.jackson.databind.ObjectMapper;

ObjectMapper mapper = new ObjectMapper();
System.out.println(mapper.writeValueAsString(obj));
```

## 取得当前的方法名
```java
Thread.currentThread().getStackTrace()[1].getMethodName()
```

## Spring 获取 HttpServletRequest
```java
HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.currentRequestAttributes()).getRequest();
```

## SpringSecurity 获取登陆用户名
```java
String username = SecurityContextHolder.getContext().getAuthentication().getName();
```

## Servlet 获取客户端的 IP
```java
/**
 * 获取客户端的 IP
 * @param request
 * @return 客户端的 IP
 */
public static String getClientIp(HttpServletRequest request) {
    final String UNKNOWN = "unknown";

    // 需要注意有多个 Proxy 的情况: X-Forwarded-For: client, proxy1, proxy2
    // 没有最近的 Proxy 的 IP, 只有 1 个 Proxy 时是客户端的 IP
    String ip = request.getHeader("X-Forwarded-For");

    if (ip == null || ip.length() == 0 || UNKNOWN.equalsIgnoreCase(ip)) {
        ip = request.getHeader("X-Real-IP");
    }

    if (ip == null || ip.length() == 0 || UNKNOWN.equalsIgnoreCase(ip)) {
        ip = request.getHeader("Proxy-Client-IP");
    }

    if (ip == null || ip.length() == 0 || UNKNOWN.equalsIgnoreCase(ip)) {
        ip = request.getHeader("WL-Proxy-Client-IP");
    }

    if (ip == null || ip.length() == 0 || UNKNOWN.equalsIgnoreCase(ip)) {
        ip = request.getHeader("HTTP_CLIENT_IP");
    }

    if (ip == null || ip.length() == 0 || UNKNOWN.equalsIgnoreCase(ip)) {
        ip = request.getHeader("HTTP_X_FORWARDED_FOR");
    }

    if (ip == null || ip.length() == 0 || UNKNOWN.equalsIgnoreCase(ip)) {
        ip = request.getRemoteAddr(); // 没有使用 Proxy 时是客户端的 IP, 使用 Proxy 时是最近的 Proxy 的 IP
    }

    return ip;
}
```

> Nginx 的配置参考:
> 
> ```
  location / {
      proxy_redirect   off;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_pass http://app_server;
      add_header Cache-Control 'no-store';
  }
  ```

> 参考 [HTTP 请求头中的 X-Forwarded-For](https://imququ.com/post/x-forwarded-for-header-in-http.html)
> 
> `X-Forwarded-For` 是用于记录代理信息的，每经过一级代理(匿名代理除外)，代理服务器都会把这次请求的来源 IP 追加在 X-Forwarded-For 中，例如来自 4.4.4.4 的一个请求，header 包含这样一行 `X-Forwarded-For: 1.1.1.1, 2.2.2.2, 3.3.3.3` 代表请求由 1.1.1.1 发出，经过三层代理，第一层是 2.2.2.2，第二层是 3.3.3.3，而本次请求的来源 IP 4.4.4.4 是第三层代理。
>
> `X-Real-IP`，一般只记录`真实发出请求的客户端 IP`，上面的例子，如果配置了 X-Read-IP，将会是 X-Real-IP: 1.1.1.1 1所以 ，如果只有一层代理，这两个头的值就是一样的

## 依赖
* Apache Commons Lang3: `org.apache.commons:commons-lang3:3.4`
* Jackson: `com.fasterxml.jackson.core:jackson-databind:2.7.3`
