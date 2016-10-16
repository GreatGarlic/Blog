---
title: Web 项目框架
date: 2016-10-15 11:10:37
tags: Spring-Web
---
项目结构:

```
├── build.gradle
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── xtuer
    │   │           └── controller
    │   │               └── DemoController.java
    │   ├── resources
    │   │   └── config
    │   │       └── spring-mvc.xml
    │   └── webapp
    │       └── WEB-INF
    │           ├── static
    │           │   ├── css
    │           │   ├── img
    │           │   │   └── favicon.ico
    │           │   ├── js
    │           │   └── lib
    │           │       ├── jquery.js
    │           │       └── template.js
    │           ├── view
    │           │   └── fm
    │           │       └── hello.fm
    │           └── web.xml
    └── test
        ├── java
        └── resources
```

<!--more-->

目录或文件       | 说明
-------------- | --------------
spring-mvc.xml | SpringMVC 的配置文件
static/js      | 存放我们自己的 js
static/css     | 存放我们自己的 css
static/img     | 存放我们自己的图片
static/lib     | 存放第三方的前端文件 js，css 等
view/fm        | 存放 Freemarker 的模版文件，后缀名为 .fm

## build.gradle
```groovy
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
    httpPort    = 8080
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
    servlet:    '3.1.0',
    fastjson:   '1.2.17',
    freemarker: '2.3.23',
    junit:      '4.12'
]

dependencies {
    compile(
            "org.springframework:spring-webmvc:$versions.spring",          // Spring MVC
            "org.springframework:spring-context-support:$versions.spring",
            "com.alibaba:fastjson:$versions.fastjson",                     // JSON
            "org.freemarker:freemarker:$versions.freemarker"               // Freemarker
    )

    compileOnly("javax.servlet:javax.servlet-api:$versions.servlet")
    testCompile("junit:junit:$versions.junit")
}
```

## DemoController.java
```java
package com.xtuer.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class DemoController {
    @GetMapping("/")
    @ResponseBody
    public String index() {
        return "Welcome";
    }

    @GetMapping("/hello")
    public String hello(ModelMap model) {
       model.put("name", "Biao");

        return "hello.fm";
    }
}
```

## hello.fm
```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title>Demo</title>
        <script src="${static}/lib/jquery.js" charset="utf-8"></script>
    </head>
    <body>
        Welcome ${name!"Guest"} to SpringMVC!
    </body>
</html>
```

> 静态文件的访问路径通过变量 `${static}` 来控制，这样就比较灵活，在不同的环境下可以把静态文件放到不同的地方，例如放到项目里，或者有单独的静态文件服务器，CDN 等。

## spring-mvc.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!-- ⾃自动扫描的包名:
         在包名 com.xtuer.controller 下的标记为 @Controller, @Service, @Component 的类
         都会⾃动的生成一个对象存储到 Spring Container ⾥
    -->
    <context:component-scan base-package="com.xtuer.controller"/>

    <!-- 默认的注解映射支持 -->
    <mvc:annotation-driven/>

    <!-- Freemarker 视图解析器 -->
    <bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
        <property name="prefix" value=""/>
        <property name="order"  value="0"/>
        <property name="cache"  value="true"/>
        <property name="contentType" value="text/html; charset=UTF-8"/>
    </bean>

    <!-- Freemarker 文件放在目录 WEB-INF/view/ftl 下 -->
    <bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
        <property name="templateLoaderPath" value="/WEB-INF/view/fm/"/>
        <property name="freemarkerSettings">
            <props>
                <prop key="defaultEncoding">UTF-8</prop>
            </props>
        </property>

        <!-- 定义变量, 在模版里直接可以使用 -->
        <property name="freemarkerVariables">
            <map>
                <entry key="static" value=""/>
            </map>
        </property>
    </bean>

    <!-- 对静态资源的访问，如 js, css, jpg, png -->
    <!-- 如 HTML 里访问 /js/jquery.js, 则实际访问的是 /WEB-INF/asset/js/jquery.js -->
    <mvc:resources mapping="/js/**"  location="/WEB-INF/static/js/"  cache-period="31556926"/>
    <mvc:resources mapping="/css/**" location="/WEB-INF/static/css/" cache-period="31556926"/>
    <mvc:resources mapping="/img/**" location="/WEB-INF/static/img/" cache-period="31556926"/>
    <mvc:resources mapping="/lib/**" location="/WEB-INF/static/lib/" cache-period="31556926"/>
    <mvc:resources mapping="/favicon.ico" location="/WEB-INF/static/img/favicon.ico" cache-period="31556926"/>
</beans>
```

## web.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
            http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1"
         metadata-complete="true">
    <!-- POST 请求的编码 -->
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

## 测试
1. 访问 <http://localhost:8080>

    > 输出: Welcome
2. 访问 <http://localhost:8080/hello>

    > 输出: Welcome Biao to SpringMVC!
