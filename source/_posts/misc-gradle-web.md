---
title: Gradle 创建 Web 项目 + Greety
date: 2016-06-29 10:30:08
tags: [Misc, Gradle]
---

项目目录结构 (asset, view, config 目录是为了管理文件而添加的，不是必须的):  
![](/img/misc/Gradle-Web.png)

<!--more-->

## 1. build.gradle

使用 Greety 启动嵌入式 Tomcat 或者 Jetty，具体参考
<http://blog.csdn.net/xiejx618/article/details/38322537>

> 不配置 servletContainer, 默认为 'jetty9'  
> 这个值可以是 'jetty7', 'jetty8', 'jetty9', 'tomcat7', 'tomcat8'

```java
group 'com.xtuer'
version '1.0-SNAPSHOT'

apply plugin: 'java'
apply plugin: 'maven'
apply plugin: 'war'
apply plugin: 'org.akhikhl.gretty'

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'org.akhikhl.gretty:gretty:1.2.4'
    }
}

gretty {
    port = 8080
    contextPath = '/'
    servletContainer = 'tomcat7'
    
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

ext {
    jstlVersion = '1.2'
    servletVersion = '3.1.0'
    springVersion = '4.1.1.RELEASE'
}

dependencies {
    compile(
            "org.springframework:spring-webmvc:$springVersion",
            "javax.servlet:jstl:$jstlVersion"
    )

    providedCompile("javax.servlet:javax.servlet-api:$servletVersion")
    testCompile "junit:junit:4.12"
}
```

## 2. web.xml
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
    
    <!-- Context Param -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            <!--classpath:spring-security.xml-->
        </param-value>
    </context-param>

    <!-- Listener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- Filter -->

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

    <!-- 默认的错误处理页面 -->
    <!--<error-page>-->
        <!--<error-code>403</error-code>-->
        <!--<location>/403</location>-->
    <!--</error-page>-->
    <!--<error-page>-->
        <!--<error-code>404</error-code>-->
        <!--<location>/404</location>-->
    <!--</error-page>-->
</web-app>
```

## 3. spring-mvc.xml
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

    <!-- 视图解析器中没有⽤ suffix 是为了可以根据 suffix ⾃动选择视图解析器，为了同时支持多种视图解析器 -->
    <!-- JSP 视图解析器，JSP 文件放在目录 WEB-INF/view/jsp 下 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/view/jsp/"/>
        <property name="order" value="1"/>
    </bean>

    <!--<mvc:default-servlet-handler/> 我们不使用默认的 servlet-handler 处理资源访问 -->
    <!-- 对静态资源的访问，如 js, css, jpg, png -->
    <!-- 如 HTML 里访问 /js/jquery.js, 则实际访问的是 /WEB-INF/resources/js/jquery.js -->
    <mvc:resources mapping="/js/**" location="/WEB-INF/resources/js/" cache-period="31556926"/>
    <mvc:resources mapping="/css/**" location="/WEB-INF/resources/css/" cache-period="31556926"/>
    <mvc:resources mapping="/image/**" location="/WEB-INF/resources/image/" cache-period="31556926"/>
</beans>
```

## 4. HelloController
```java
package com.xtuer.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class HelloController {
    @RequestMapping("hello")
    @ResponseBody
    public String hello() {
        return "Hello --";
    }
}
```



## 5. 启动 Tomcat 并测试
1. 进入项目目录，执行命令 `gradle run`

    ```
    可以使用参数设置日志级别
    -d, --debug: Log in debug mode (includes normal stacktrace)
    -i, --info: Set log level to info
    -q, --quiet: Log errors only.
    使用 gradle --help 查看 gradle 的参数说明
    ```
2. 访问 <http://localhost:8080/hello>
3. 页面输出: `Hello --`
4. Greety 能实现自动热部署，不需要使用 SpringLoaded 或者 JRebel 等插件。例如在 HelloController 里新创建一个函数，可以看到马上就被自动加载了

## 6. 使用 Debug 模式启动 Tomcat
执行 `gradle debug`，使用 `suspend` 的模式监听 `5005` 端口

## 7. Greety 的命令
[Gretty tasks](http://webcache.googleusercontent.com/search?q=cache:oswRB8fKZCoJ:akhikhl.github.io/gretty-doc/Gretty-tasks+&cd=1&hl=de&ct=clnk&gl=us)，截图为部分:

> keypress 为按下任意键就结束 Tomcat  
> appStop 指在终端输入 gradle appStop 结束 Tomcat，或者 按下 Ctrl + C 结束 Tomcat

![](/img/misc/Greety-Commands.png)
