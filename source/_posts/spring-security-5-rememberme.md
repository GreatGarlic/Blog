---
title: Spring Security Remember Me
date: 2016-04-10 10:59:20
tags: Spring-Security
---

## 什么是 Remember Me？
* 访问 <http://biao.com/admin> 
* 登录成功
* 重启浏览器
* 再次访问 <http://biao.com/admin>
* 需要重新登录

如果启用了 `Remember Me`，登录后重启浏览器访问 <http://biao.com/admin> 就不需要重新登录了。

<!--more-->

给 Spring Security 添加 Remember Me 功能，只需要 2 步:

* 在登录的 form 表单里添加

    ```html
    <input type="checkbox" name="remember-me"/> Remember Me<br>
    ```
* 在 Spring Security 配置文件的 http 元素下添加(2592000 为 30 天: 24 \* 3600 \* 30)

    ```xml
    <remember-me key="uniqueAndSecret" token-validity-seconds="2592000"/>
    ```

## Login.htm
```html
<html>
<head>
    <title>Login Page</title>
</head>
<body>
    ${error!}${logout!}
    <form name="loginForm" action="/login" method="POST">
        Username: <input type="text" name="username" /><br>
        Password: <input type="password" name="password" /><br>
        <input type="checkbox" name="remember-me"/> Remember Me<br>
        <input name="submit" type="submit" value="登陆" />
    </form>
</body>
</html>
```

## spring-security.xml
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
        <intercept-url pattern="/admin" access="hasRole('ADMIN')"/>
        <intercept-url pattern="/login" access="permitAll"/>

        <form-login login-page="/login"
                    login-processing-url="/login"
                    default-target-url  ="/hello"
                    authentication-failure-url="/login?error=1"
                    username-parameter="username"
                    password-parameter="password"/>
        <access-denied-handler error-page="/deny"/>
        <logout logout-url="/logout" logout-success-url="/login?logout=1"/>

        <csrf disabled="true"/>
        <remember-me key="uniqueAndSecret" token-validity-seconds="2592000"/>
    </http>

    <beans:bean id="userDetailsService" class="com.xtuer.service.MyUserDetailsService"/>
    <authentication-manager>
        <authentication-provider user-service-ref="userDetailsService">
            <password-encoder hash="bcrypt"/>
        </authentication-provider>
    </authentication-manager>
</beans:beans>
```

## 测试
* 访问 <http://biao.com/admin> 
* 登录成功
* 重启浏览器
* 再次访问 <http://biao.com/admin>
* 不需要重新登录
