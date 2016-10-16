---
title: 乱码处理
date: 2016-10-15 22:48:18
tags: Spring-Web
---
字符集主要涉及 2 个方面

* 文件本身的字符集（文件，数据库存储使用，返回给浏览器端的 html 内容）
* 程序中编码解码时候使用的字符集（如解析 http 请求的数据）

为了防止乱码，我们规定：`所有的字符集都用 UTF-8`。

<!--more-->

## 1. IDEA 设置字符集
`IDEA Encoding`, `Project Encoding`, `Properties Encoding` 都使用 UTF-8，这样 IDEA 里创建的文件都是 UTF-8 编码的。
![](/img/spring-web/messy-code.png)

## 2. JSP 中字符集的设置
```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
```

## 3. HTML 中字符集的设置
```html
<head>
    <meta http-equiv="content-type" content="text/html; charset=UTF-8"/>
</head>
```

## 4. Tomcat 处理 GET 请求的字符集设置
前台网页的 GET 请求以 UTF-8 来解析，`pom.xml` 里设置 uriEncoding

```xml
<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443"
           URIEncoding="UTF-8"/>
```

## 5. Tomcat 处理 POST 请求的字符集设置
前台网页的 POST 请求以 UTF-8 来解析，`web.xml` 加上字符集的 filter 处理 POST 的中文

```xml
<!-- 处理 POST 的中文 -->
<filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
        <param-name>forceEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

## 6. Freemarker 模版生成文件的字符集设置
在配置 Freemarker 的时候已经设置过了。

```xml
<property name="contentType" value="text/html; charset=UTF-8"/>
```

## 7. AJAX 返回有乱码?
由处理 `@ResponseBody` 返回字符串的 MessageConverter 的编码设置造成的，配置 `spring-mvc.xml` 中的 MessageConverter（去掉 ~~<mvc:annotation-driven/>~~）

```xml
<!-- 默认的注解映射支持 -->
<!--<mvc:annotation-driven/>-->

<mvc:annotation-driven>
   <mvc:message-converters register-defaults="true">
       <bean class="org.springframework.http.converter.StringHttpMessageConverter">
           <constructor-arg value="UTF-8" />
       </bean>
   </mvc:message-converters>
</mvc:annotation-driven>
```

## 8. 数据库的字符集设置
创建数据库的时候，选择 Encoding 为 UTF-8

## 9. 配置 JDBC 连接数据库的字符集为 UTF-8
```
jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=UTF-8
```

## 10. 使用 UTF-8 启动 Web Server(Tomcat)
### 10.1. Unix Like 配置 Tomcat
在 Tomcat 启动文件 `catalina.sh` 中加一个 `-Dfile.encoding=UTF-8`

```
JAVA_OPTS="$JAVA_OPTS -Dfile.encoding=UTF-8"
```

### 10.2. Windows 配置 Tomcat
在 Tomcat 启动文件 `catalina.bat` 中加一个 `-Dfile.encoding=UTF-8`

```
set "JAVA_OPTS=%JAVA_OPTS% -Dfile.encoding=UTF-8"
```
