---
title: Spring Security Login Util
date: 2016-04-25 13:37:32
tags: Spring-Security
---

查看用户是否登陆，调用 `SecurityUtil.isLogin()` 即可。

```java
import org.springframework.security.core.context.SecurityContextHolder;

public class SecurityUtil {
    /**
     * 判断当前用户是否已经登陆
     * @return 登陆状态返回 true, 否则返回 false
     */
    public static boolean isLogin() {
        String username = SecurityContextHolder.getContext().getAuthentication().getName();
        return !"anonymousUser".equals(username);
    }
}
```
