---
title: Spring Security Login Util
date: 2016-04-25 13:37:32
tags: SpringSecurity
---

查看用户是否登陆，调用 `SecurityUtils.isLogin()` 即可。

```java
import org.springframework.security.core.context.SecurityContextHolder;

public class SecurityUtils {
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

> **getAuthentication()** 返回的是一个 Authentication 对象，未登陆时它的 **getPrinciple()** 返回的是字符串 **anonymousUser**，登陆后返回的是 **UserDetailsService.loadUserByUsername()** 返回的对象，也即是说，我们可以实现一个 User 类，继承自 org.springframework.security.core.userdetails.User，在我们实现的 User 类中保存用户数据，在 loadUserByUsername() 中返回此用户对象，在其他地方就可以使用下面这样的代码获取登录用户的信息:
>
```java
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
```
<!--more-->

下面提供一个 User 类作为参考:

```java
package com.xtuer.bean;

import lombok.Getter;
import lombok.Setter;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;

import java.util.Arrays;
import java.util.HashSet;
import java.util.Set;

@Getter
@Setter
public class User extends org.springframework.security.core.userdetails.User {
    private Long id;
    private String username;
    private String password;
    private Set<String> roles = new HashSet<>();

    public User() {
        // 父类不允许空的用户名、密码和权限，所以给个默认的，这样就可以用默认的构造函数创建 User 对象。
        super("non-exist-username", "", new HashSet<GrantedAuthority>());
    }

    public User(String username, String password, String... roles) {
        super(username, password, buildAuthorities(roles));

        this.username = username;
        this.password = password;
        this.roles.addAll(Arrays.asList(roles));
    }

    /**
     * 使用 new User() 创建的对象，或者 username、password 和 roles 改变后没有在父类中更新对应数据，
     * 可以使用这个方法重新构造一个正确的 User 对象，这个对象是为了和 Spring Security 一起使用。
     */
    public static User userWithAuthorities(User user) {
        return new User(user.getUsername(), user.getPassword(), user.getRoles().toArray(new String[0]));
    }

    /**
     * 使用字符串数组的 roles 创建 GrantedAuthority 的 Set。
     */
    public static Set<GrantedAuthority> buildAuthorities(String... roles) {
        Set<GrantedAuthority> authorities = new HashSet<>();

        for (String role : roles) {
            authorities.add(new SimpleGrantedAuthority(role));
        }

        return authorities;
    }
}
```

```java
package com.xtuer.security;

import com.xtuer.bean.User;
import com.xtuer.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

@Service
public class UserDetailsService implements org.springframework.security.core.userdetails.UserDetailsService {
    @Autowired
    private UserService userService;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userService.findUserByUsername(username);

        if (user == null) {
            throw new UsernameNotFoundException(username + " not found!");
        }

        return user;
    }
}
```

