---
title: Spring Security 用户信息数据源
date: 2016-04-10 10:22:02
tags: Spring-Security
---

前面章节中用户名、密码、权限都是写在配置文件里的，不能动态的管理用户的权限，大多数时候显然是不行的。这里介绍从其他数据源读取用户的信息，例如从数据库，LDAP 等。只需要给 `authentication-provider` 提供接口 `UserDetailsService` 的实现类即可，使用这个类获取用户的信息:

* 修改 spring-security.xml 中的 authentication-provider
* 接口 UserDetailsService 的实现类 MyUserDetailsService
* 类 User
* 类 UserRole
* 类 UserDao
* 其他文件和前面的一样

<!--more-->

## spring-security.xml
> 修改 `authentication-manager`

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

    <beans:bean id="userDetailsService" class="com.xtuer.service.MyUserDetailsService"/>
    <authentication-manager>
        <authentication-provider user-service-ref="userDetailsService"/>
    </authentication-manager>
</beans:beans>
```

## MyUserDetailsService
分 2 步：身份验证 (Authentication)，权限验证 (Authorization)

* 用户在登录页面输入 username, password，然后点击登录按钮
* 方法 loadUserByUsername() 使用 username 查找到用户的信息，如密码，权限等
* 使用查找到的密码和用户输入的密码比较，如果相等，则身份验证成功
* 身份验证成功后，使用用户的权限和页面的访问权限比较，页面的权限配置在 `intercept-url`

```java
package com.xtuer.service;

import com.xtuer.dao.UserDao;
import com.xtuer.domain.UserRole;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;

import java.util.ArrayList;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

public class MyUserDetailsService implements UserDetailsService {
    private UserDao userDao = new UserDao();

    /**
     * 使用 username 加载用户的信息，如密码，权限等
     * @param username 登陆表单中用户输入的用户名
     * @return
     * @throws UsernameNotFoundException
     */
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        com.xtuer.domain.User user = userDao.findUserByUsername(username);

        if (user == null) {
            throw new UsernameNotFoundException(username + " not found!");
        }

        return buildUserDetails(user);
    }

    /**
     * Converts User user to org.springframework.security.core.userdetails.User
     * @param user
     * @return
     */
    private User buildUserDetails(com.xtuer.domain.User user) {
        List<GrantedAuthority> authorities = buildUserAuthorities(user.getUserRoles());
        return new User(user.getUsername(), user.getPassword(), user.isEnabled(), true, true, true, authorities);
    }

    /**
     * 把用户的权限 UserRole 转换成 GrantedAuthority
     * @param userRoles 用户的权限
     * @return
     */
    private List<GrantedAuthority> buildUserAuthorities(Set<UserRole> userRoles) {
        Set<GrantedAuthority> authorities = new HashSet<GrantedAuthority>();

        // Build user's authorities
        for (UserRole userRole : userRoles) {
            authorities.add(new SimpleGrantedAuthority(userRole.getRole()));
        }

        return new ArrayList<GrantedAuthority>(authorities);
    }
}
```

## User
```java
package com.xtuer.domain;

import java.util.HashSet;
import java.util.Set;

public class User {
    private int id;
    private String username;
    private String password;
    private boolean enabled;
    private Set<UserRole> userRoles = new HashSet<UserRole>();

    public User() {

    }

    public User(String username, String password, boolean enabled, Set<UserRole> userRoles) {
        this.username = username;
        this.password = password;
        this.enabled = enabled;
        this.userRoles = userRoles;
    }

    // Getters and setters
}
```

## UserRole
```java
package com.xtuer.domain;

public class UserRole {
    private int id;
    private String role;

    public UserRole() {

    }

    public UserRole(String role) {
        this.role = role;
    }

    // Getters and setters
}
```

## UserDao
```java
package com.xtuer.dao;

import com.xtuer.domain.User;
import com.xtuer.domain.UserRole;

import java.util.*;

public class UserDao {
    private static Map<String, User> users = new HashMap<String, User>();

    static {
        // 模拟数据源，可以是多种，如数据库，LDAP，从配置文件读取等
        UserRole userRole = new UserRole("ROLE_USER");   // 普通用户权限
        UserRole adminRole = new UserRole("ROLE_ADMIN"); // 管理员权限

        users.put("admin", new User("admin", "Passw0rd", true, new HashSet<UserRole>(Arrays.asList(adminRole))));
        users.put("alice", new User("alice", "Passw0rd", true, new HashSet<UserRole>(Arrays.asList(userRole))));
    }

    public User findUserByUsername(String username) {
        return users.get(username);
    }
}
```

## 测试
* 访问 http://biao.com/hello
* 访问 http://biao.com/admin
* 输入错误的用户名或密码，观察登陆失败的页面
* 输入正确的用户名和密码，继续登陆
* 访问 http://biao.com/logout，观察注销成功的页面

