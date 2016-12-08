---
title: Spring Security 入门
date: 2016-04-09 22:34:34
tags: [Java, Cas]
---

## 目录结构

```
├── main
│   ├── java
│   │   └── com
│   │       └── xtuer
│   │           └── controller
│   │               └── HelloController.java
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

Spring Security + CAS 的依赖有:

* spring-security-web
* spring-security-config
* spring-security-cas

> 注意: Spring Security 的版本和 Spring 的版本不是一样的

## build.gradle

```
group 'com.xtuer'
version '1.0'

apply plugin: 'java'
apply plugin: 'maven'
apply plugin: 'war'
apply plugin: 'org.akhikhl.gretty'

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'org.akhikhl.gretty:gretty:1.4.0'
    }
}

gretty {
    port = 8081
    contextPath = ''
    servletContainer = 'tomcat7'

    inplaceMode  = 'hard'
    debugSuspend = false
    managedClassReload      = true
    recompileOnSourceChange = false
}

tasks.withType(JavaCompile) {
    sourceCompatibility = JavaVersion.VERSION_1_8
    targetCompatibility = JavaVersion.VERSION_1_8
}

[compileJava, compileTestJava, javadoc]*.options*.encoding = 'UTF-8'

////////////////////////////////////////////////////////////////////////////////
//                                   Maven 依赖                               //
////////////////////////////////////////////////////////////////////////////////
repositories {
    mavenLocal()
    mavenCentral()
}

ext.versions = [
    spring:     '4.3.0.RELEASE',
    springSecurity: '4.0.4.RELEASE',
    servlet:    '3.1.0',
    fastjson:   '1.2.17',
    freemarker: '2.3.23',
    junit:      '4.12'
]

dependencies {
    compile(
            "org.springframework:spring-webmvc:$versions.spring",             // Spring MVC
            "org.springframework:spring-context-support:$versions.spring",
            "org.springframework.security:spring-security-web:$versions.springSecurity", // Spring Security
            "org.springframework.security:spring-security-config:$versions.springSecurity",
            "org.springframework.security:spring-security-cas:$versions.springSecurity",
            "com.alibaba:fastjson:$versions.fastjson",  // JSON
            "org.freemarker:freemarker:$versions.freemarker" // Freemarker
    )

    compileOnly("javax.servlet:javax.servlet-api:$versions.servlet")
    testCompile("org.springframework:spring-test:$versions.spring")
    testCompile("junit:junit:$versions.junit")
}
```

## spring-security.xml

> 权限定义时为 `ROLE_ADMIN`，判断是否有权限时使用 `hasRole('ADMIN')`  
> 需要多个权限: `hasRole('ADMIN') and hasRole('DBA')`

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

    <http auto-config="true">
        <intercept-url pattern="/admin" access="hasRole('ADMIN')" />
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
        <h2>Welcome : ${username}</h2>
    </#if>
</body>
</html>
```

## 测试
* 访问 <http://www.xtuer.com:8081/hello>，因为没有权限要求，正常访问页面
* 访问 <http://www.xtuer.com:8081/admin>，因为需要 admin 的权限，所以会自动 redirect 到 <http://www.xtuer.com:8081/login> 进行登录
    * 用户名输入 `admin`，密码输入 `Passw0rd`，点击 Login，登录成功，自动 redirect 到登陆前先前的页面 <http://www.xtuer.com:8081/admin>
    * 用户名输入 `alice`，密码输入 `Passw0rd`，点击 Login，登录成功，提示无权限访问

## 参考
* [Spring Security 4 Tutorial](http://websystique.com/spring-security-tutorial/)
* [Spring Security 4 Hello World Annotation+XML ](http://websystique.com/spring-security/spring-security-4-hello-world-annotation-xml-example/)
