---
title: Spring Security 入门
date: 2016-04-09 22:34:34
tags: Spring-Security
---

## 目录结构
```
├── main
│   ├── java
│   │   └── com
│   │       └── xtuer
│   │           └── controller
│   │               ├── HelloController.java
│   │               └── LoginController.java
│   ├── resources
│   │   └── config
│   │       ├── spring-mvc.xml
│   │       └── spring-security.xml
│   └── webapp
│       └── WEB-INF
│           ├── view
│           │   └── fm
│           │       ├── admin.htm
│           │       └── hello.htm
│           └── web.xml
└── test
    ├── java
    └── resources
```

<!--more-->

## Gradle 依赖
Spring Security 的依赖有:

* spring-security-web
* spring-security-config

> 注意: Spring Security 的版本和 Spring 的版本不是一样的

```
ext {
    springVersion   = '4.2.5.RELEASE'
    springSecurityVersion = '4.0.4.RELEASE'
    jstlVersion     = '1.2'
    servletVersion  = '3.1.0'
    jacksonVersion  = '2.5.3'
    freemarkerVersion = '2.3.20'
}

dependencies {
    compile(
            "org.springframework:spring-webmvc:$springVersion",             // Spring MVC
            "org.springframework.security:spring-security-web:$springSecurityVersion", // Spring Security
            "org.springframework.security:spring-security-config:$springSecurityVersion",
            "org.springframework:spring-context-support:$springVersion",
            "com.fasterxml.jackson.core:jackson-databind:$jacksonVersion",  // JSON
            "org.freemarker:freemarker:$freemarkerVersion"                  // Freemarker
    )

    compile("javax.servlet:jstl:$jstlVersion") // JSTL
    compileOnly("javax.servlet:javax.servlet-api:$servletVersion")
}
```

## spring-security.xml
> 权限定义为 `ROLE_ADMIN`，判断是否有权限使用 `hasRole('ADMIN') 或者 hasRole('ROLE_ADMIN') `，前缀 ROLE_ 可以省略  
> 需要多个权限: `hasRole('ADMIN') and hasRole('DBA')`  
> 有任意一个权限: `hasAnyRole('ROLE_ADMIN', 'ROLE_DBA')`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans
        xmlns="http://www.springframework.org/schema/security"
        xmlns:beans="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/security
            http://www.springframework.org/schema/security/spring-security.xsd">
    <http security="none" pattern="/static/**"/>
    <http auto-config="true">
        <intercept-url pattern="/admin" access="hasRole('ROLE_ADMIN')" />
    </http>

    <authentication-manager>
        <authentication-provider>
            <user-service>
                <user name="admin" password="Passw0rd" authorities="ROLE_ADMIN"/>
                <user name="alice" password="Passw0rd" authorities="ROLE_USER"/>
            </user-service>
        </authentication-provider>
    </authentication-manager>
</beans:beans>
```

## web.xml

> Spring Security 是使用 Servlet Filter 来实现的，filter-name 必须为 `springSecurityFilterChain`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app
        xmlns="http://java.sun.com/xml/ns/javaee"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
            http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
        version="3.0"
        metadata-complete="false">
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            classpath:config/spring-security.xml
        </param-value>
    </context-param>

    <!-- Spring Security Filter -->
    <filter>
        <filter-name>springSecurityFilterChain</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy
        </filter-class>
    </filter>
    <filter-mapping>
        <filter-name>springSecurityFilterChain</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!-- Spring MVC Servlet -->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:config/spring-mvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

## LoginController
```java
package com.xtuer.controller;

import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.web.authentication.logout.SecurityContextLogoutHandler;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Controller
public class LoginController {
    // 注销处理函数
    @RequestMapping(value="/logout", method = RequestMethod.GET)
    @ResponseBody
    public String logoutPage(HttpServletRequest request, HttpServletResponse response) {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();

        if (auth != null){
            new SecurityContextLogoutHandler().logout(request, response, auth);
        }

        return "Logout success";
    }
}
```

## HelloController
```java
package com.xtuer.controller;

import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class HelloController {
    @RequestMapping("/")
    @ResponseBody
    public String index() {
        return "index page";
    }

    @RequestMapping(value = {"/hello"}, method = RequestMethod.GET)
    public String welcomePage(ModelMap model) {
        model.addAttribute("title", "Spring Security Hello World");
        model.addAttribute("message", "This is welcome page!");

        return "hello.htm";
    }

    @RequestMapping(value = "/admin", method = RequestMethod.GET)
    public String adminPage(ModelMap model) {
        model.addAttribute("title", "Spring Security Hello World");
        model.addAttribute("message", "This is protected page!");

        UserDetails userDetails = (UserDetails) SecurityContextHolder.getContext().getAuthentication().getPrincipal();
        model.addAttribute("username", userDetails.getUsername());

        return "admin.htm";
    }
}
```

## hello.htm
```html
<html>
<body>
    <h1>Title : ${title}</h1>
    <h1>Message : ${message}</h1>
</body>
</html>
```

## admin.htm
```htm
<html>
<body>
    <h1>Title : ${title}</h1>
    <h1>Message : ${message}</h1>

    <#if username??>
        <h2>Welcome : ${username} <a href="/logout">Logout</a></h2>
    </#if>
</body>
</html>
```

## 测试
* 访问 <http://biao.com/hello>，因为没有权限要求，正常访问页面
* 访问 <http://biao.com/admin>，因为需要 admin 的权限，所以会自动 redirect 到 <http://biao.com/login> 进行登录
    * 用户名输入 `admin`，密码输入 `Passw0rd`，点击 Login，登录成功，自动 redirect 到登陆前先前的页面 <http://biao.com/admin>
    * 用户名输入 `alice`，密码输入 `Passw0rd`，点击 Login，登录成功，提示无权限访问

## 参考
* [Spring Security 4 Tutorial](http://websystique.com/spring-security-tutorial/)
* [Spring Security 4 Hello World Annotation+XML ](http://websystique.com/spring-security/spring-security-4-hello-world-annotation-xml-example/)
