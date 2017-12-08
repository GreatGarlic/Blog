---
title: Spring Security 自定义登陆表单
date: 2016-04-09 23:40:58
tags: SpringSecurity
---

虽然 Spring Security 提供了默认的登录表单，实际项目里肯定是不可以直接使用的，当然 Spring Security 也提供了自定义登录表单的功能。

<!--more-->

## spring-security.xml
| Config                             | Description                              |
| ---------------------------------- | ---------------------------------------- |
| login-page                         | GET: 登陆页面的 URL                           |
| login-processing-url               | POST: 登陆表单提交的 URL                        |
| default-target-url                 | 直接访问 login 页面登陆成功后重定向的 URL               |
| authentication-success-handler-ref | Reference to an **AuthenticationSuccessHandler** bean which should be used to handle a successful authentication request. Should not be used in combination with default-target-url (or always-use-default-target-url) as the implementation should always deal with navigation to the subsequent destination. 同一个登陆页面，管理员和普通用户登陆后重定向到不同页面时可用. |
| authentication-failure-url         | 登陆失败重定向的 URL                             |
| error-page                         | 访问权限不够时的 URL                             |
| logout-url                         | 注销的 URL                                  |
| logout-success-url                 | 注销成功的 URL                                |
| username-parameter                 | 登陆表单中用户名的 input 的 name                   |
| password-parameter                 | 登陆表单中密码的 input 的 name                    |
| `<csrf disabled="true"/>`          | 禁用 csrf，默认是启用的，如果想使用 csrf，需要在登陆表单里加上这样的语句:<br> `<input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/>` |
| ooooooooooooooooooooooooooooo      |                                          |

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
        <intercept-url pattern="/admin" access="hasRole('ADMIN')"/>
        <intercept-url pattern="/login" access="permitAll"/>
        <form-login login-page="/login"
                    login-processing-url="/login"
                    default-target-url  ="/hello"
                    authentication-failure-url="/login?error"
                    username-parameter="username"
                    password-parameter="password"/>
        <access-denied-handler error-page="/deny" />
        <logout logout-url="/logout" logout-success-url="/login?logout" />
        <csrf disabled="true"/>
    </http>

    <authentication-manager>
        <authentication-provider>
            <user-service>
                <user name="admin" password="{noop}Passw0rd" authorities="ROLE_ADMIN"/>
                <user name="alice" password="{noop}Passw0rd" authorities="ROLE_USER"/>
            </user-service>
        </authentication-provider>
    </authentication-manager>
</beans:beans>
```

> Spring Security 允许多个 `<http>` 存在:
>
> 上面的配置 `<intercept-url pattern="/login" access="permitAll"/>`，导致访问 /login 页面时也要经过很多 filter，例如 UsernamePasswordAuthenticationFilter, SecurityContextPersistenceFilter, SessionRepositoryFilter 等，其实不需要登陆就能访问的页面，例如登陆页面, js, css, png 等是不需要经过这些 filter 的，则可以配置 http 的 security 为 none，这样就能提高程序的效率:
```xml
<http security="none" pattern="/page/login"/>
<http security="none" pattern="/static/**"/>
```

## LoginController

前一节中，注销是在 LoginController 中使用函数 `logoutPage()` 来处理的，这一节注销直接在 spring-security.xml 里配置 `<logout>` 来实现的，把 `logoutPage()` 函数从 LoginController 里删除

```java
package com.xtuer.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.*;

@Controller
public class LoginController {
    @GetMapping("/login")
    public String loginPage(@RequestParam(value="error", required=false) String error,
                            @RequestParam(value="logout", required=false) String logout,
                            ModelMap model) {
        if (error != null) {
            model.put("error", "Username or password is not correct");
        } else if (logout != null) {
            model.put("logout", "Logout successful");
        }

        return "login.html";
    }

    @RequestMapping("/deny")
    @ResponseBody
    public String denyPage() {
        return "You have no permission to access the page";
    }
}
```

## 登陆表单 login.html

```html
<html>
<head>
    <title>Login Page</title>
</head>
<body>
    <span th:text="${error}" th:if="${error} != null"></span>
    <span th:text="${logout}" th:if="${logout} != null"></span>

    <form name="loginForm" action="/login" method="POST">
        Username: <input type="text" name="username" /><br>
        Password: <input type="password" name="password" /><br>
        <input name="submit" type="submit" value="登陆" />
    </form>
</body>
</html>
```

## 测试
* 访问 <http://localhost:8080/hello>
* 访问 <http://localhost:8080/admin>
* 输入错误的用户名或密码，观察登陆失败的页面
* 输入正确的用户名和密码 admin/Passw0rd，继续登陆
* 访问 <http://localhost:8080/logout>，观察注销成功的页面
* 访问 <http://localhost:8080/admin>
* 输入正确的用户名和密码 alice/Passw0rd，继续登陆
* 提示权限不够


## 自定义登陆成功的 handler

设置 authentication-success-handler-ref 为自定义登陆成功的 handler，就可以使用同一个登录表单，登录成功后根据用户的角色或者权限重定向到不同的页面:

```xml
<beans:bean id="loginSuccessHandler" class="com.xtuer.security.LoginSuccessHandler"/>

<http security="none" pattern="/static/**"/>
<http auto-config="true">
    <intercept-url pattern="/admin" access="hasRole('ADMIN')"/>
    <intercept-url pattern="/login" access="permitAll"/>

    <form-login login-page="/login"
                login-processing-url="/login"
                default-target-url  ="/"
                authentication-success-handler-ref="loginSuccessHandler"
                authentication-failure-url="/login?error"
                username-parameter="username"
                password-parameter="password"/>
    <access-denied-handler error-page="/deny" />

    <logout logout-url="/logout" logout-success-url="/login?logout" />
    <csrf disabled="true"/> <!-- 去掉这一行不行 -->
</http>
```
> 注意：自定义登录成功的 handler 后 `default-target-url` 不再起作用。

```java
package com.xtuer.security;

import org.springframework.security.core.Authentication;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.web.authentication.AuthenticationSuccessHandler;
import org.springframework.security.web.savedrequest.HttpSessionRequestCache;
import org.springframework.security.web.savedrequest.SavedRequest;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class LoginSuccessHandler implements AuthenticationSuccessHandler {
    @Override
    public void onAuthenticationSuccess(HttpServletRequest request,
                                        HttpServletResponse response,
                                        Authentication authentication) throws IOException, ServletException {
        // 用户登录成功后访问不同的页面:
        // 1. 获取用户登录前访问的 URL
        // 2. 如果 url 不为空，访问登录前的页面
        // 3. 如果 url 为空，登录后根据用户的角色访问对应的页面
        SavedRequest savedRequest = new HttpSessionRequestCache().getRequest(request, response);
        String redirectUrl = (savedRequest == null) ? "" : savedRequest.getRedirectUrl();

        if (!redirectUrl.isEmpty()) {
            response.sendRedirect(redirectUrl);
            return;
        }

        // 获取登录用户信息
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();

        for (GrantedAuthority authority : auth.getAuthorities()) {
            String role = authority.getAuthority(); // 用户的权限

            // 不同的用户访问不同的页面
            if ("ROLE_ADMIN".equals(role)) {
                redirectUrl = "/admin";
            } else {
                redirectUrl = "/hello";
            }
        }

        response.sendRedirect(redirectUrl);
    }
}
```

