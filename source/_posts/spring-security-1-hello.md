---
title: Spring Security 入门
date: 2016-04-09 22:34:34
tags: SpringSecurity
---

## 目录结构
```
├── build.gradle
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── xtuer
    │   │           └── controller
    │   │               └── HelloController.java
    │   ├── resources
    │   │   ├── config
    │   │   │   ├── application-servlet.xml
    │   │   │   └── spring-security.xml
    │   │   └── logback.xml
    │   └── webapp
    │       └── WEB-INF
    │           ├── page
    │           │   ├── admin.html
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

## Gradle 依赖
Spring Security 的依赖有:

* spring-security-web
* spring-security-config

> 注意: Spring Security 的版本和 Spring 的版本不是一样的

```
ext.versions = [
    spring   : '5.0.2.RELEASE',
    springSecurity: '5.0.0.RELEASE',
    servlet  : '4.0.0',
    fastjson : '1.2.41',
    thymeleaf: '3.0.9.RELEASE',
    lombok   : '1.16.18',
    logback  : '1.2.3',
    jclOverSlf4j: '1.7.25'
]

dependencies {
    compile(
            "org.springframework:spring-webmvc:${versions.spring}",
            "org.springframework:spring-context-support:${versions.spring}",
            "org.springframework.security:spring-security-web:${versions.springSecurity}",
            "org.springframework.security:spring-security-config:${versions.springSecurity}",
            "com.alibaba:fastjson:${versions.fastjson}",
            "org.thymeleaf:thymeleaf:${versions.thymeleaf}",
            "org.thymeleaf:thymeleaf-spring5:${versions.thymeleaf}",
            "ch.qos.logback:logback-classic:${versions.logback}", // Logback
            "org.slf4j:jcl-over-slf4j:${versions.jclOverSlf4j}"
    )

    compileOnly("javax.servlet:javax.servlet-api:${versions.servlet}")
    compileOnly("org.projectlombok:lombok:${versions.lombok}")
    testCompile("org.springframework:spring-test:${versions.spring}")
}
```

## spring-security.xml

> 权限定义为 `ROLE_ADMIN`，判断是否有权限使用 `hasRole('ADMIN') 或者 hasRole('ROLE_ADMIN') `，前缀 ROLE_ 可以省略  
> 需要多个权限: `hasRole('ADMIN') and hasRole('DBA')`  
> 有任意一个权限: `hasAnyRole('ROLE_ADMIN', 'ROLE_DBA')`
>
> 如果想使用权限列表的方式，而不是上面的这种表达式，需要设置 `<http auto-config="true" use-expressions="false">`，然后才能  
> `<intercept-url pattern="/demo/filters" access="ROLE_USER"/>`

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
                <!-- 密码都是 Passw0rd，不是 {noop}Passw0rd，前缀 {noop} 是 Spring Security 5 用来确定 PasswordEncoder 的 -->
                <user name="admin" password="{noop}Passw0rd" authorities="ROLE_ADMIN"/>
                <user name="alice" password="{noop}Passw0rd" authorities="ROLE_USER"/>
            </user-service>
        </authentication-provider>
    </authentication-manager>
</beans:beans>
```

> auto-config: A legacy attribute which automatically registers a login form, BASIC authentication and a logout URL and logout services. If unspecified, defaults to "false". We'd recommend you    avoid using this and instead explicitly configure the services you require.
>
> 参考 FormLoginConfigurer, DefaultLoginPageGeneratingFilter and AbstractAuthenticationProcessingFilter.

## application-servlet.xml

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
    <!-- 控制器 -->
    <context:component-scan base-package="com.xtuer.controller"/>

    <!-- 注解映射支持 -->
    <mvc:annotation-driven>
        <!--enableMatrixVariables="true">-->
        <mvc:message-converters register-defaults="true">
            <!-- StringHttpMessageConverter 编码为 UTF-8，防止乱码 -->
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
        <property name="prefix"       value="/WEB-INF/page/"/>
        <property name="templateMode" value="HTML"/>
        <property name="cacheable"    value="false"/>
    </bean>

    <bean id="templateEngine" class="org.thymeleaf.spring5.SpringTemplateEngine">
        <property name="templateResolver" ref="templateResolver"/>
    </bean>

    <bean class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
        <property name="templateEngine"    ref="templateEngine"/>
        <property name="characterEncoding" value="UTF-8"/>  <!--解决中文乱码-->
    </bean>

    <!-- 静态资源的访问，如 js, css, jpg, png -->
    <!-- 如 HTML 里访问 /static/js/jquery.js, 则实际访问的是 /WEB-INF/static/js/jquery.js -->
    <mvc:resources mapping="/static/js/**"   location="/WEB-INF/static/js/"   cache-period="31556926"/>
    <mvc:resources mapping="/static/css/**"  location="/WEB-INF/static/css/"  cache-period="31556926"/>
    <mvc:resources mapping="/static/img/**"  location="/WEB-INF/static/img/"  cache-period="31556926"/>
    <mvc:resources mapping="/static/lib/**"  location="/WEB-INF/static/lib/"  cache-period="31556926"/>
    <mvc:resources mapping="/static/html/**" location="/WEB-INF/static/html/" cache-period="31556926"/>
    <mvc:resources mapping="/favicon.ico"    location="/WEB-INF/static/img/favicon.ico" cache-period="31556926"/>
</beans>
```

## web.xml

> Spring Security 是使用 Servlet Filter 来实现的，filter-name 必须为 `springSecurityFilterChain`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
         http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1"
         metadata-complete="true">
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
            <param-value>classpath:config/application-servlet.xml</param-value>
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

        return "hello.html";
    }

    @RequestMapping(value = "/admin", method = RequestMethod.GET)
    public String adminPage(ModelMap model) {
        model.addAttribute("title", "Spring Security Hello World");
        model.addAttribute("message", "This is protected page!");

        UserDetails userDetails = (UserDetails) SecurityContextHolder.getContext().getAuthentication().getPrincipal();
        model.addAttribute("username", userDetails.getUsername());

        return "admin.html";
    }
}
```

## hello.html

```html
<html>
<body>
    <h1 th:text="|Title : ${title}|"></h1>
    <h1 th:text="|Message : ${message}|"></h1>
</body>
</html>
```

## admin.html

```htm
<html>
<body>
    <h1 th:text="|Title : ${title}|"></h1>
    <h1 th:text="|Message : ${message}|"></h1>

    <h2 th:if="${username} != ''"><span th:text="|Welcome : ${username}|"></span> <a href="/logout">Logout</a></h2>
</body>
</html>
```

## 测试

* 访问 <http://localhost:8080/hello>，因为没有权限要求，正常访问页面
* 访问 <http://localhost:8080/admin>，因为需要 admin 的权限，所以会自动 redirect 到 <http://localhost:8080/login> 进行登录
    * 用户名输入 `admin`，密码输入 `Passw0rd`，点击 Login，登录成功，自动 redirect 到登陆前先前的页面 <http://localhost:8080/admin>
    * 用户名输入 `alice`，密码输入 `Passw0rd`，点击 Login，登录成功，提示无权限访问

## 参考
* [Spring Security 4 Tutorial](http://websystique.com/spring-security-tutorial/)
* [Spring Security 4 Hello World Annotation+XML ](http://websystique.com/spring-security/spring-security-4-hello-world-annotation-xml-example/)
