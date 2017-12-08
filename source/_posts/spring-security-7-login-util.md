---
title: Spring Security Login Util
date: 2016-04-25 13:37:32
tags: SpringSecurity
---

查看当前登录用户的信息，调用 `SecurityUtils.getLoginUser()` 即可:

```java
package com.xtuer.utils;

import com.xtuer.bean.User;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;

public final class SecurityUtils {
    /**
     * 获取登陆用户的信息
     *
     * @return 返回登陆的用户，如果没有登陆则返回 null
     */
    public static User getLoginUser() {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();

        // URL 没有经过 Spring Security 登陆验证的 filter 时 auth 为 null
        if (auth == null) {
            return null;
        }

        Object p = auth.getPrincipal();

        return p instanceof User ? (User) p : null;
    }
}
```

> **getAuthentication()** 返回的是一个 Authentication 对象，未登陆时它的 **getPrinciple()** 返回的是字符串 **anonymousUser**，登陆后返回的是 **UserDetailsService.loadUserByUsername()** 返回的对象，也即是说，我们可以实现一个 User 类，继承自 org.springframework.security.core.userdetails.User，在我们实现的 User 类中保存用户数据，在 loadUserByUsername() 中返回此用户对象。


User 类的实现请参考[用户信息数据源一章](/spring-security-3-datasource)。