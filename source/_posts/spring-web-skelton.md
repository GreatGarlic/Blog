---
title: 项目框架
date: 2016-10-15 11:10:37
tags: SpringWeb
---
## 工程结构

```
├── build.gradle
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── xtuer
    │   │           ├── bean
    │   │           ├── controller
    │   │           │   └── HelloController.java
    │   │           └── service
    │   ├── resources
    │   │   └── config
    │   │       ├── application.properties
    │   │       └── springmvc-servlet.xml
    │   └── webapp
    │       └── WEB-INF
    │           ├── page
    │           │   └── hello.html
    │           ├── static
    │           │   ├── css
    │           │   ├── img
    │           │   ├── js
    │           │   └── lib
    │           └── web.xml
    └── test
        ├── java
        └── resources
```

<!--more-->

| 目录或文件                  | 说明                                     |
| ---------------------- | -------------------------------------- |
| build.gradle           | Gradle 的工程文件                           |
| application.properties | 关键配置的文件，例如数据库的配置，Redis 的配置等            |
| springmvc-servlet.xml  | Spring MVC 的配置文件                       |
| resources/config       | 存放配置文件的目录                              |
| static/js              | 存放我们自己的 js                             |
| static/css             | 存放我们自己的 css                            |
| static/img             | 存放我们自己的图片                              |
| static/lib             | 存放第三方的前端文件 js，css 等，例如 jQuery，CKEditor |
| page                   | 模版文件的目录                                |
| controller             | 控制器的目录                                 |

## build.gradle

> 注意: 使用 UTF-8 运行 Gradle: Gradle 的使用请参考 http://qtdebug.com/tags/Gradle/
>
> 注意: 升级 Gretty 使用的 spring loaded:
>
> Gretty 默认使用的 spring loaded 版本比较低，Spring 4 的时候没有问题，和 Spring 5 一起使用时由于使用了 Lambda 导致类加载器出错，需要升级到 [spring loaded 1.2.8](http://www.mvnrepository.com/artifact/org.springframework/springloaded/1.2.8.RELEASE) 解决这个问题(jvmArgs 中配置 spring loaded jar 包的路径):
>
```
gretty {
    ...
    jvmArgs = ['-javaagent:<path>/springloaded-1.2.8.RELEASE.jar', '-noverify']
}
```

```groovy
plugins {
    id 'war'
    id 'java'
    id 'org.akhikhl.gretty' version '2.0.0'
}

gretty {
    httpPort = 8080
    contextPath = ''
    servletContainer = 'tomcat8'

    inplaceMode  = 'hard'
    debugSuspend = false
    managedClassReload      = true
    recompileOnSourceChange = true
    jvmArgs = ['-javaagent:/Users/Biao/Documents/springloaded-1.2.8.RELEASE.jar', '-noverify']
}

////////////////////////////////////////////////////////////////////////////////
//                                   Maven 依赖                               //
////////////////////////////////////////////////////////////////////////////////
repositories {
    mavenCentral()
}

ext.versions = [
    spring   : '5.0.2.RELEASE',
    servlet  : '4.0.0',
    fastjson : '1.2.41',
    thymeleaf: '3.0.9.RELEASE'
]

dependencies {
    compile(
            "org.springframework:spring-webmvc:${versions.spring}",
            "org.springframework:spring-context-support:${versions.spring}",
            "com.alibaba:fastjson:${versions.fastjson}",
            "org.thymeleaf:thymeleaf-spring5:${versions.thymeleaf}"
    )

    compileOnly("javax.servlet:javax.servlet-api:${versions.servlet}")
    testCompile("org.springframework:spring-test:${versions.spring}")
}

////////////////////////////////////////////////////////////////////////////////
//                                    JVM                                     //
////////////////////////////////////////////////////////////////////////////////
sourceCompatibility = JavaVersion.VERSION_1_8
targetCompatibility = JavaVersion.VERSION_1_8
[compileJava, compileTestJava, javadoc]*.options*.encoding = 'UTF-8'

tasks.withType(JavaCompile) {
    options.compilerArgs << '-Xlint:unchecked' << '-Xlint:deprecation'
}
```

## HelloController.java

```java
package com.xtuer.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import java.util.HashMap;
import java.util.Map;

@Controller
public class HelloController {
    /**
     * http://localhost:8080/page/hello
     */
    @GetMapping("/page/hello")
    public String hello(ModelMap model) {
        model.put("name", "Biao");

        return "hello.html";
    }

    /**
     * http://localhost:8080/api/json
     */
    @GetMapping("/api/json")
    @ResponseBody
    public Object json() {
        Map<String, String> map = new HashMap<>();
        map.put("name", "Alice");
        map.put("sock", "24");

        return map;
    }
}
```

## hello.html

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>Solo</title>
</head>

<body>
    Hello <span th:text="${name}"/>, you are welcome!
</body>

</html>
```

## springmvc-servlet.xml

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
    <!-- 配置文件 -->
    <context:property-placeholder location="classpath:config/application.properties"/>

    <!-- 控制器 -->
    <context:component-scan base-package="com.xtuer.controller"/>

    <!-- 注解映射支持 -->
    <mvc:annotation-driven>
        <mvc:message-converters>
            <!-- StringHttpMessageConverter 编码为UTF-8，防止乱码 -->
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                <constructor-arg value="UTF-8"/>
                <property name="supportedMediaTypes">
                    <list>
                        <bean class="org.springframework.http.MediaType">
                            <constructor-arg index="0" value="text"/>
                            <constructor-arg index="1" value="plain"/>
                            <constructor-arg index="2" value="UTF-8"/>
                        </bean>
                        <bean class="org.springframework.http.MediaType">
                            <constructor-arg index="0" value="*"/>
                            <constructor-arg index="1" value="*"/>
                            <constructor-arg index="2" value="UTF-8"/>
                        </bean>
                    </list>
                </property>
            </bean>
            <!-- FastJson -->
            <bean class="com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter4">
                <property name="supportedMediaTypes">
                    <list>
                        <value>text/html;charset=UTF-8</value>
                        <value>application/json;charset=UTF-8</value>
                    </list>
                </property>
                <property name="fastJsonConfig">
                    <bean class="com.alibaba.fastjson.support.config.FastJsonConfig">
                        <property name="features">
                            <list>
                                <value>AllowArbitraryCommas</value>
                                <value>AllowUnQuotedFieldNames</value>
                                <value>DisableCircularReferenceDetect</value>
                            </list>
                        </property>
                        <property name="dateFormat" value="yyyy-MM-dd HH:mm:ss"/>
                    </bean>
                </property>
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>

    <!-- 使用 thymeleaf 模版 -->
    <bean id="templateResolver" class="org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver">
        <property name="prefix"       value="/WEB-INF/page/"/> <!-- 模版文件放在 page 目录下 -->
        <property name="templateMode" value="HTML"/>
        <property name="cacheable"    value="${thymeleafCacheable}"/> <!-- cacheable 线上环境用 true，开发环境用 false -->
    </bean>
    <bean id="templateEngine" class="org.thymeleaf.spring5.SpringTemplateEngine">
        <property name="templateResolver" ref="templateResolver"/>
    </bean>
    <bean class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
        <property name="templateEngine"    ref="templateEngine"/>
        <property name="characterEncoding" value="UTF-8"/>
    </bean>

    <!-- 静态资源的访问，如 js, css, jpg, png -->
    <!-- 如 HTML 里访问 /static/js/foo.js, 则实际访问的是 /WEB-INF/static/js/foo.js -->
    <mvc:resources mapping="/static/js/**"   location="/WEB-INF/static/js/"   cache-period="31556926"/>
    <mvc:resources mapping="/static/lib/**"  location="/WEB-INF/static/lib/"  cache-period="31556926"/>
    <mvc:resources mapping="/static/css/**"  location="/WEB-INF/static/css/"  cache-period="31556926"/>
    <mvc:resources mapping="/static/img/**"  location="/WEB-INF/static/img/"  cache-period="31556926"/>
    <mvc:resources mapping="/static/html/**" location="/WEB-INF/static/html/" cache-period="31556926"/>
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
            <param-value>classpath:config/springmvc-servlet.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

## 启动 Web 服务

命令行 cd 进入工程所在目录，也就是 build.gradle 的目录，然后执行 `gradle clean appStart` 就启动了嵌入式的 Web 服务器 Tomcat，可以访问 Web 服务了。

## 测试

1. 访问 <http://localhost:8080/page/hello>

    > 输出: Hello Biao, you are welcome!

2. 修改 hello.html，刷新页面，马上就能看到更新后的内容，不需要重启 Web 服务器

3. 访问 <http://localhost:8080/api/json>

    > 输出: {"sock":"24","name":"Alice"}

4. 修改 HelloController 中的 Java 代码，例如 `map.put("sock", "24777")`，刷新页面，很快就能看到更新后的内容，不需要重启 Web 服务器(修改 Java 代码热更新的时间会长一点，一般 2 秒左右)

## 总结

本章介绍了 Web 工程的目录结构和基础配置，以及热更新的使用加速开发效率，点击 [web-1.7z](/download/web-1.7z) 可下载工程源码。