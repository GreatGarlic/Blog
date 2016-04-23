---
title: Spring Security 自定义登陆表单
date: 2016-04-09 23:40:58
tags: Spring-Security
---

虽然 Spring Security 提供了默认的登录表单，实际项目里肯定是不可以直接使用的，当然 Spring Security 也提供了自定义登录表单的功能。

<!--more-->

## spring-security.xml
| Config | Description |
| ------ | ----------- |
| login-page | 登陆页面的 URL |
| login-processing-url | 登陆表单提交的 URL |
| default-target-url | 直接访问 login 页面登陆成功后重定向的 URL |
| authentication-failure-url | 登陆失败重定向的 URL |
| error-page | 访问权限不够时的 URL |
| logout-url | 注销的 URL |
| logout-success-url | 注销成功的 URL |
| username-parameter | 登陆表单中用户名的 input 的 name |
| password-parameter | 登陆表单中密码的 input 的 name |
| `<csrf disabled="true"/>` | 禁用 csrf，默认是启用的，如果想使用 csrf，需要在登陆表单里加上这样的语句:<br> `<input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/>` |

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
        <access-denied-handler error-page="/deny" />
        <logout logout-url="/logout" logout-success-url="/login?logout=1" />

        <csrf disabled="true"/>
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

## LoginController
前一节中，注销是在 LoginController 中使用函数 `logoutPage()` 来处理的，这一节注销直接在 spring-security.xml 里配置 `<logout>` 来实现的，把 `logoutPage()` 函数从 LoginController 里删除

```java
package com.xtuer.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class LoginController {
    @RequestMapping(value="/login", method = RequestMethod.GET)
    public String loginPage(@RequestParam(value="error", required=false) String error,
                            @RequestParam(value="logout", required=false) String logout,
                            ModelMap model) {
        if (error != null) {
            model.put("error", "Username or password is not correct");
        } else if (logout != null) {
            model.put("logout", "Logout successful");
        }

        return "login.htm";
    }

    @RequestMapping("/deny")
    @ResponseBody
    public String denyPage() {
        return "You have no permission to access the page";
    }
}
```

## 登陆表单 login.htm

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
        <input name="submit" type="submit" value="登陆" />
    </form>
</body>
</html>
```

## 测试
* 访问 <http://biao.com/hello>
* 访问 <http://biao.com/admin>
* 输入错误的用户名或密码，观察登陆失败的页面
* 输入正确的用户名和密码，继续登陆
* 访问 <http://biao.com/logout>，观察注销成功的页面

