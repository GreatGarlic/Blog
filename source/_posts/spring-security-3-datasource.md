---
title: Spring Security 用户信息数据源
date: 2016-04-10 10:22:02
tags: SpringSecurity
---

前面章节中用户名、密码、权限都是写在配置文件里的，不能动态的管理用户的权限，大多数时候显然是不行的。这里介绍从其他数据源读取用户的信息，例如从数据库，LDAP 等。只需要给 `authentication-provider` 提供接口 `UserDetailsService` 的实现类即可，使用这个类获取用户的信息，涉及以下内容:

* 修改 spring-security.xml 中的 authentication-provider
* 类 UserDetailsService 实现了 Spring Security 的接口 UserDetailsService
* 类 User
* 类 UserService

<!--more-->

## spring-security.xml
> 修改 `authentication-manager` 下的 **user-service-ref** 为我们自定义的 **UserDetailsService**

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
        <csrf disabled="true"/>
    </http>

    <beans:bean id="userDetailsService" class="com.xtuer.security.UserDetailsService"/>
    <authentication-manager>
        <authentication-provider user-service-ref="userDetailsService"/>
    </authentication-manager>
</beans:beans>
```

## UserDetailsService

UserDetailsService 的作用是根据登录表单中用户的用户名查找用户信息用于身份认证。Spring Security 中授权分 2 步：身份验证 (Authentication)，权限验证 (Authorization)

* 用户在登录页面输入 username, password，然后点击登录按钮
* 方法 loadUserByUsername() 使用 username 查找到用户的信息，如密码，权限等
* Spring Security 使用查找到的密码和加密后用户输入的密码进行比较，如果相等，则身份验证成功
* 身份验证成功后，使用用户的权限和页面的访问权限比较，页面的权限配置在 `intercept-url`

```java
package com.xtuer.security;

import com.xtuer.bean.User;
import com.xtuer.service.UserService;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UsernameNotFoundException;

public class UserDetailsService implements org.springframework.security.core.userdetails.UserDetailsService {
    private UserService userService = new UserService();

    /**
     * 使用 username 加载用户的信息，如密码，权限等
     * @param  username 登陆表单中用户输入的用户名
     * @return 返回查找到的用户对象
     * @throws UsernameNotFoundException
     */
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

## UserService

UserService 用户查找用户信息

```java
package com.xtuer.service;

import com.xtuer.bean.User;

import java.util.HashMap;
import java.util.Map;

public class UserService {
    private static Map<String, User> users = new HashMap<String, User>();

    static {
        // 模拟数据源，可以是多种，如数据库，LDAP，从配置文件读取等
        users.put("admin", new User("admin", "{noop}Passw0rd", "ROLE_ADMIN"));
        users.put("alice", new User("alice", "{noop}Passw0rd", "ROLE_USER"));
    }

    public User findUserByUsername(String username) {
        return users.get(username);
    }
}
```

## User

```java
package com.xtuer.bean;

import com.alibaba.fastjson.JSON;
import lombok.Getter;
import lombok.Setter;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.AuthorityUtils;

import java.util.*;

/**
 * 用户类型，根据 userdetails.User 的设计，roles, authorities, expired 等状态不能修改，
 * 只能是创建用户对象的时候传入进来。
 */
@Getter
@Setter
public class User extends org.springframework.security.core.userdetails.User {
    private Long   id;
    private String username;
    private String password;
    private String mail;
    private boolean enabled;
    private Set<String> roles = new HashSet<>(); // 用户的角色

    public User() {
        // 父类不允许空的用户名、密码和权限，所以给个默认的，这样就可以用默认的构造函数创建 User 对象。
        super("non-exist-username", "", new HashSet<>());
    }

    /**
     * 使用账号、密码、角色创建用户
     *
     * @param username 账号
     * @param password 密码
     * @param roles    角色
     */
    public User(String username, String password, String... roles) {
        this(username, password, true, roles);
    }

    /**
     * 使用账号、密码、是否禁用、角色创建用户
     *
     * @param username 账号
     * @param password 密码
     * @param enabled  是否禁用
     * @param roles    角色
     */
    public User(String username, String password, boolean enabled, String... roles) {
        super(username, password, enabled, true, true, true, AuthorityUtils.createAuthorityList(roles));
        this.username = username;
        this.password = password;
        this.enabled  = enabled;
        this.roles.addAll(Arrays.asList(roles));
    }

    /**
     * 用户信息修改后，例如角色修改后不会更新到父类的 authorities 中，需要重新创建一个用户对象才行
     *
     * @param user 已有用户对象
     * @return 新的用户对象，权限等信息更新到了父类的 authorities 中
     */
    public static User userForSpringSecurity(User user) {
        return new User(user.username, user.password, user.enabled, user.getRoles().toArray(new String[0]));
    }

    public static void main(String[] args) {
        User user1 = new User();
        System.out.println(JSON.toJSONString(user1));
        System.out.println(user1.getRoles());

        User user2 = new User("Bob", "Passw0rd", "ROLE_USER", "ROLE_ADMIN");
        System.out.println(JSON.toJSONString(user2));
        System.out.println(user2.getRoles());
    }
}
```

## 测试
* 访问 http://localhost:8080/hello
* 访问 http://localhost:8080/admin
* 输入错误的用户名或密码，观察登陆失败的页面
* 输入正确的用户名和密码，继续登陆
* 访问 <http://localhost:8080/logout>，观察注销成功的页面

