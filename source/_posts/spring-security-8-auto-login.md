---
title: Spring Security 自动登录
date: 2017-02-27 14:43:09
tags: Spring-Security
---
前面的实现可以使用表单进行登陆了，但是某些时候需要自动登录，例如使用 QQ 的第三方登录，服务器收到登陆成功的回调后，需要在我们的系统中继续使用本地账号登陆才行，这时就会需要实现自动登录的功能，还有使用 AJAX 等也不能使用表单登陆，也是需要调用登陆的接口才可以。

为了提供登陆的接口，需要实现 **AuthenticationProvider** 进行登陆，不再使用 Spring Security 的默认实现。当然实现了自动登录后，并不会影响 Spring Security 的表单登陆。
<!--more-->

项目的文件如下，需要增加修改的文件有:

* MyUserDetails.java 
* MyUserDetailsService.java
* MyAuthenticationProvider.java
* SecurityUtils.java
* UserDao.java
* AutoLoginController.java
* spring-security.xml
* login.fm

```
├── java
│   └── com
│       └── xtuer
│           ├── bean
│           │   ├── User.java
│           │   └── UserRole.java
│           ├── controller
│           │   ├── AutoLoginController.java
│           │   ├── HelloController.java
│           │   └── LoginController.java
│           ├── dao
│           │   └── UserDao.java
│           ├── security
│           │   ├── MyAuthenticationProvider.java
│           │   ├── MyUserDetails.java
│           │   ├── MyUserDetailsService.java
│           │   └── SecurityUtils.java
│           └── service
├── resources
│   └── config
│       ├── spring-mvc.xml
│       └── spring-security.xml
└── webapp
    └── WEB-INF
        ├── static
        ├── view
        │   ├── admin.fm
        │   ├── hello.fm
        │   └── login.fm
        └── web.xml
```

## MyUserDetails.java
自定义的 UserDetails 作为 Principle 保存到 Spring Security 的登陆信息里，这样就可以使用自定义的 UserDetails 根据我们的业务规则保存足够的信息，默认的 Principle 只保存了用户名，绝大多数时候只有用户名是不够的。

```java
package com.xtuer.security;

import com.xtuer.bean.User;
import com.xtuer.bean.UserRole;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;

import java.util.ArrayList;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

/**
 * 自定义的 UserDetails 可以保存用户的其他信息到 session 里, 例如 user id 等.
 */
public class MyUserDetails extends org.springframework.security.core.userdetails.User {
    private int userId;

    public MyUserDetails(User user) {
        super(user.getUsername(), user.getPassword(), user.isEnabled(), true, true, true, buildUserAuthorities(user));
        this.userId = user.getId();
    }

    public int getUserId() {
        return userId;
    }

    public void setUserId(int userId) {
        this.userId = userId;
    }

    /**
     * 把用户的权限 UserRole 转换成 GrantedAuthority
     * @param user 用户
     * @return
     */
    private static List<GrantedAuthority> buildUserAuthorities(User user) {
        Set<UserRole> userRoles = user.getUserRoles();
        Set<GrantedAuthority> authorities = new HashSet<GrantedAuthority>();

        // Build user's authorities
        for (UserRole userRole : userRoles) {
            authorities.add(new SimpleGrantedAuthority(userRole.getRole()));
        }

        return new ArrayList<GrantedAuthority>(authorities);
    }
}
```

## MyUserDetailsService.java
使用用户名从数据库查找到用户的信息，然后构建 UserDetails。

```java
package com.xtuer.security;

import com.xtuer.dao.UserDao;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;

public class MyUserDetailsService implements UserDetailsService {
    private UserDao userDao = new UserDao();

    /**
     * 使用 username 加载用户的信息，如密码，权限等
     *
     * @param username 登陆表单中用户输入的用户名
     * @return
     */
    @Override
    public UserDetails loadUserByUsername(String username) {
        com.xtuer.bean.User user = userDao.findUserByUsername(username);

        if (user == null) {
            System.out.println(username + " not found!");
        }

        return user == null ? null : new MyUserDetails(user);
    }
}
```

> [The UserDetailsService](http://docs.spring.io/spring-security/site/docs/3.0.x/reference/technical-overview.html#d4e731)
>
```java
Object principal = SecurityContextHolder.getContext().getAuthentication().getPrincipal();
if (principal instanceof UserDetails) {
    String username = ((UserDetails)principal).getUsername();
} else {
    String username = principal.toString();
}
```
> Another item to note from the above code fragment is that you can obtain a principal from the Authentication object. The principal is just an Object. Most of the time this can be cast into a UserDetails object. UserDetails is a central interface in Spring Security. It represents a principal, but in an extensible and application-specific way. Think of UserDetails as the adapter between your own user database and what Spring Security needs inside the SecurityContextHolder. Being a representation of something from your own user database, quite often you will cast the UserDetails to the original object that your application provided, so you can call business-specific methods (like getEmail(), getEmployeeNumber() and so on).
>
>By now you're probably wondering, so when do I provide a UserDetails object? How do I do that? I thought you said this thing was declarative and I didn't need to write any Java code - what gives? The short answer is that there is a special interface called UserDetailsService. The only method on this interface accepts a String-based username argument and returns a UserDetails:
>
>UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
This is the most common approach to loading information for a user within Spring Security and you will see it used throughout the framework whenever information on a user is required.
>
>On successful authentication, UserDetails is used to build the Authentication object that is stored in the SecurityContextHolder (more on this below). The good news is that we provide a number of UserDetailsService implementations, including one that uses an in-memory map (InMemoryDaoImpl) and another that uses JDBC (JdbcDaoImpl). Most users tend to write their own, though, with their implementations often simply sitting on top of an existing Data Access Object (DAO) that represents their employees, customers, or other users of the application. Remember the advantage that whatever your UserDetailsService returns can always be obtained from the SecurityContextHolder using the above code fragment.

## MyAuthenticationProvider.java
MyAuthenticationProvider 提供了登陆的逻辑。

```java
package com.xtuer.security;

import org.springframework.security.authentication.*;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.password.PasswordEncoder;

import javax.annotation.Resource;

public class MyAuthenticationProvider implements AuthenticationProvider {
    @Resource(name="userDetailsService")
    private MyUserDetailsService userDetailsService;

    @Resource(name="passwordEncoder")
    private PasswordEncoder passwordEncoder;

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        // [1] token 中的用户名和密码都是用户输入的，不是数据库里的
        UsernamePasswordAuthenticationToken token = (UsernamePasswordAuthenticationToken) authentication;
        // [2] 使用用户名从数据库读取用户信息
        UserDetails userDetails = userDetailsService.loadUserByUsername(token.getName());

        // [3] 检查用户信息
        if(userDetails == null) {
            throw new UsernameNotFoundException("用户不存在");
        } else if (!userDetails.isEnabled()){
            throw new DisabledException("用户已被禁用");
        } else if (!userDetails.isAccountNonExpired()) {
            throw new AccountExpiredException("账号已过期");
        } else if (!userDetails.isAccountNonLocked()) {
            throw new LockedException("账号已被锁定");
        } else if (!userDetails.isCredentialsNonExpired()) {
            throw new LockedException("凭证已过期");
        }

        // [4] 根据不同的情况比对密码
        if (isOAuthUser(userDetails)) {
            // 通过 OAuth 登陆过来的，例如密码是调用 SecurityHelper.login() 前判断是 OAuth 合法登陆的，
            // 然后就查询数据库得到本地用户的密码，这里可以只需要和数据库里的使用等于比较就可以了，具体的仍然需要根据业务逻辑调整
            // 如果不涉及到 OAuth 用户登陆，可以删除此段代码
        } else {
            String encryptedPassword = userDetails.getPassword();   // 数据库用户的密码，一般都是加密过的
            String inputPassword = (String) token.getCredentials(); // 用户输入的密码

            // 根据加密算法加密用户输入的密码，然后和数据库中保存的密码进行比较
            if(!passwordEncoder.matches(inputPassword, encryptedPassword)) {
                throw new BadCredentialsException("用户名/密码无效");
            }
        }

        // [5] 成功登陆，把用户信息提交给 Spring Security
        // 把 userDetails 作为 principal 的好处是可以放自定义的 UserDetails，这样可以存储更多有用的信息，而不只是 username，
        // 默认只有 username，这里的密码使用数据库中保存的密码，而不是用户输入的明文密码，否则就暴露了密码的明文
        return new UsernamePasswordAuthenticationToken(userDetails, userDetails.getPassword(), userDetails.getAuthorities());
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return UsernamePasswordAuthenticationToken.class.equals(authentication);
    }

    /**
     * 判断是否通过 OAuth 登陆的用户，例如 QQ，优酷等都提供了 OAuth 第三方登陆服务。
     *
     * @param userDetails
     * @return 如果是 OAuth 用户则返回 true，否则返回 false
     */
    private boolean isOAuthUser(UserDetails userDetails) {
        // 以 QQ_ 开头的用户名，说明是使用 QQ 的 OAuth 登陆的用户
        // 以 YK_ 开头的用户名，说明是使用优酷的 OAuth 登陆的用户
        // ...
        return userDetails.getUsername().startsWith("QQ_");
    }
}
```

## SecurityUtils.java
SecurityUtils 提供了登陆的接口 `login()`，还有一些和登陆有关的方法。

```java
package com.xtuer.security;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.web.authentication.rememberme.TokenBasedRememberMeServices;
import org.springframework.security.web.savedrequest.HttpSessionRequestCache;
import org.springframework.security.web.savedrequest.SavedRequest;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.annotation.Resource;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class SecurityUtils {
    @Resource(name = "authenticationManager")
    private AuthenticationManager authenticationManager;

    @Autowired
    private TokenBasedRememberMeServices tokenBasedRememberMeServices;

    /**
     * 判断当前用户是否已经登陆
     * @return 登陆状态返回 true, 否则返回 false
     */
    public static boolean isLogin() {
        String username = SecurityContextHolder.getContext().getAuthentication().getName();
        return !"anonymousUser".equals(username);
    }

    /**
     * 取得登陆用户的 ID, 如果没有登陆则返回 -1
     * @return 登陆用户的 ID
     */
    public static int getLoginUserId() {
        Object principle = SecurityContextHolder.getContext().getAuthentication().getPrincipal();

        if (SecurityUtils.isLogin()) {
            MyUserDetails userDetails = (MyUserDetails) principle;
            return userDetails.getUserId();
        }

        return -1;
    }

    /**
     * 登陆
     *
     * @param username
     * @param password
     * @return 登陆后需要访问的页面的 URL
     */
    public String login(String username, String password) {
        HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        HttpServletResponse response = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getResponse();

        String defaultTargetUrl = "/"; // 默认登陆成功的页面
        String redirectUrl = "/login?error=1"; // 默认为登陆错误页面

        try {
            Authentication token = new UsernamePasswordAuthenticationToken(username, password);
            token = authenticationManager.authenticate(token); // 登陆
            SecurityContextHolder.getContext().setAuthentication(token);
            tokenBasedRememberMeServices.onLoginSuccess(request, response, token); // 使用 remember me

            // 重定向到登陆前的页面
            SavedRequest savedRequest = new HttpSessionRequestCache().getRequest(request, response);
            redirectUrl = (savedRequest != null) ? savedRequest.getRedirectUrl() : defaultTargetUrl;
        } catch (Exception ex) {
            System.out.println(ex.getMessage());
        }

        return redirectUrl;
    }
}
```

## UserDao.java
增加了 QQ 用户 QQ_Admin，用户名的前缀为 QQ_，只是为了模拟第三方登录后需要使用本地账号自动登录时使用。

```java
package com.xtuer.dao;

import com.xtuer.bean.User;
import com.xtuer.bean.UserRole;

import java.util.Arrays;
import java.util.HashMap;
import java.util.HashSet;
import java.util.Map;

public class UserDao {
    private static Map<String, User> users = new HashMap<String, User>();

    static {
        // 模拟数据源，可以是多种，如数据库，LDAP，从配置文件读取等
        UserRole userRole = new UserRole("ROLE_USER");   // 普通用户权限
        UserRole adminRole = new UserRole("ROLE_ADMIN"); // 管理员权限

        // Passw0rd 的 BCrypt 加密结果
        String password = "$2a$10$gtaxGaHMfxMRj6rqK/kp0.5TPF13CBvnXhvD7teUmeftH1cX0Mb6S";
        users.put("admin", new User("admin", password, true, new HashSet<UserRole>(Arrays.asList(adminRole))));
        users.put("alice", new User("alice", password, true, new HashSet<UserRole>(Arrays.asList(userRole))));
        users.put("QQ_admin", new User("QQ_admin", password, true, new HashSet<UserRole>(Arrays.asList(adminRole))));
    }

    public User findUserByUsername(String username) {
        return users.get(username);
    }
}
```

## AutoLoginController.java
在 AutoLoginController 中实现列举了第三方登录后绑定本地用户，然后自动登录的逻辑，使用登陆接口进行登陆，AJAX 实现登陆时就可以用到，还有不存在的用户登陆。

```java
package com.xtuer.controller;

import com.xtuer.security.SecurityUtils;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

import javax.annotation.Resource;

@Controller
public class AutoLoginController {
    @Resource(name="securityUtils")
    private SecurityUtils securityUtils;

    /**
     * OAuth 用户绑定本地用户
     * 然后自动登陆
     */
    @GetMapping("/bindingUser")
    public String bindingUser() {
        // [[1]] 绑定已有用户
        // [[2]] 绑定好后进行自动登陆
        String username = "QQ_admin";
        String password = "wrong"; // OAuth 授权的用户登陆不需要密码，因为不是在我们平台登陆的

        return "redirect:" + securityUtils.login(username, password);
    }

    /**
     * 不存在的用户登陆
     */
    @GetMapping("/nonExistingUserLogin")
    public String nonExistingUserLogin() {
        String username = "nonExistingUser";
        String password = "flash";

        return "redirect:" + securityUtils.login(username, password);
    }

    /**
     * admin 登陆
     */
    @GetMapping("/adminLogin")
    public String adminLogin() {
        String username = "admin";
        String password = "Passw0rd";

        return "redirect:" + securityUtils.login(username, password);
    }
}
```

## spring-security.xml
需要注意的是和以前相比较，authentication-manager 的配置有变化，把密码加密部分去掉了，密码匹配的实现在 MyAuthenticationProvider 中使用。还因为使用了 @Resource, @Autowired 自动装配功能，所以需要把 `<context:annotation-config/>` 添加到配置中。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans
        xmlns="http://www.springframework.org/schema/security"
        xmlns:beans="http://www.springframework.org/schema/beans"
        xmlns:context="http://www.springframework.org/schema/context"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/security
            http://www.springframework.org/schema/security/spring-security.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd">
    <context:annotation-config/>

    <http auto-config="true">
        <intercept-url pattern="/login" access="permitAll"/>
        <intercept-url pattern="/admin" access="hasRole('ADMIN')"/>

        <form-login login-page="/login"
                    login-processing-url="/login"
                    default-target-url  ="/hello"
                    authentication-failure-url="/login?error=1"
                    username-parameter="username"
                    password-parameter="password"/>
        <access-denied-handler error-page="/deny" />
        <logout logout-url="/logout" logout-success-url="/login?logout=1" />

        <csrf disabled="true"/>
        <remember-me key="uniqueAndSecret" token-validity-seconds="2592000"/>
    </http>

    <beans:bean id="securityUtils" class="com.xtuer.security.SecurityUtils"/>
    <beans:bean id="passwordEncoder" class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder"/>
    <beans:bean id="userDetailsService" class="com.xtuer.security.MyUserDetailsService"/>
    <beans:bean id="authenticationProvider" class="com.xtuer.security.MyAuthenticationProvider"/>

    <authentication-manager alias="authenticationManager">
        <authentication-provider ref="authenticationProvider"/>
    </authentication-manager>
</beans:beans>
```

## login.fm
增加了几种不同的登陆链接。

```html
<html>
<head>
    <title>Login Page</title>
</head>
<body>
    ${error!}${logout!}
    <form name="loginForm" action="/login" method="POST">
        Username: <input type="text" name="username"/><br>
        Password: <input type="password" name="password"/><br>
        <input type="checkbox" name="remember-me"/> Remember Me<br>
        <input name="submit" type="submit" value="登陆"/>
    </form>

    <a href="/bindingUser">QQ 绑定用户自动登陆</a><br>
    <a href="/adminLogin">Admin 登陆</a><br>
    <a href="/nonExistingUserLogin">不存在用户登陆</a>
</body>
</html>
```

## 测试
1. 访问 <http://localhost:8080/admin>，重定向到登陆页面
2. 使用表单登陆，功能和以前一样
3. 注销，访问 <http://localhost:8080/admin>，重定向到登陆页面
4. 点击 `QQ 绑定用户自动登陆`，登陆成功后重定向到 Admin 页面
5. 注销，访问 <http://localhost:8080/admin>，重定向到登陆页面
6. 点击，`Admin 登陆`，登陆成功后重定向到 Admin 页面
7. 注销，访问 <http://localhost:8080/admin>，重定向到登陆页面
8. 点击 `不存在用户登陆`，提示登陆失败

## 项目下载
由于项目的内容比较多，所以可以下载到开发环境里测试: [spring-security.7z](/download/spring-security.7z)


