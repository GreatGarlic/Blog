---
title: Spring Security 加密密码
date: 2016-04-10 10:45:54
tags: SpringSecurity
---

明文保存密码是不可取的，可以使用 `SHA`，`BCrypt` 等对密码进行加密。

<!--more-->

BCrypt 算法与 MD5/SHA 算法有一个很大的区别，每次生成的 hash 值都是不同的，就可以免除存储 salt，暴力破解起来也更困难。BCrypt 加密后的字符长度比较长，有60位，所以用户表中密码字段的长度，如果打算采用 BCrypt 加密存储，字段长度不得低于 60。

下面的代码展示怎么使用 BCrypt 进行加密:

```java
import org.junit.Test;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

public class EncryptPassword {
    @Test
    public void encrypt() {
        PasswordEncoder passwordEncoder = new BCryptPasswordEncoder();

        for (int i = 0; i < 5; ++i) {
            // 每次生成的密码都不一样
            String encryptedPassword = passwordEncoder.encode("Passw0rd");
            System.out.println(encryptedPassword);
            System.out.println(passwordEncoder.matches("Passw0rd", encryptedPassword)); // true
            System.out.println(passwordEncoder.matches("Password", encryptedPassword)); // false
        }
    }
}
```

输出:

```
$2a$10$l7vPVeqwb9GiVjURV5J2QO1CM5qxwk00/Ra5qEog0WgP7O5XV0Ble
true
false
$2a$10$jeyMfHF88mNJb9v.mQ7YiuZ8oTU.pHaiKdT1NLOM38eXj7heHZHg2
true
false
$2a$10$ux43/3JcHUC1hszyoJaH0eQhv7LkIVfL7p1cW80WxfxeTr2dUY6kO
true
false
$2a$10$KdUmhaJOJ30klEcKiYT25.fIRPrMs4xONHOQh4JvmpKSjJ8d9.QKG
true
false
$2a$10$gQKUOoFuevnCkoej3.AvAO9YzHKCKYmKuiSfEGHL22piY2FfNDQYu
true
false
```

随意取其中任意一个都可以，因为每次生成都是不一样的，所以取第一个就可以了。

## Spring Security 使用 BCrypt 加密
* 在 spring-security.xml 里的 authentication-provider 中启用 BCrypt 加密

    ```xml
    <authentication-manager>
        <authentication-provider user-service-ref="userDetailsService">
            <password-encoder hash="bcrypt"/>
        </authentication-provider>
    </authentication-manager>
    ```
*  把 BCrypt 加密后的密码保存到数据库(怎么加密？参考上面的示例代码)

## UserDao
> 从数据源取得的密码是加密后的密码

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

        // Passw0rd 的 BCrypt 加密结果
        String password = "$2a$10$gtaxGaHMfxMRj6rqK/kp0.5TPF13CBvnXhvD7teUmeftH1cX0Mb6S";

        users.put("admin", new User("admin", password, true, new HashSet<UserRole>(Arrays.asList(adminRole))));
        users.put("alice", new User("alice", password, true, new HashSet<UserRole>(Arrays.asList(userRole))));
    }

    public User findUserByUsername(String username) {
        return users.get(username);
    }
}
```

## 测试
* 访问 http://biao.com/admin
* 输入错误的用户名或密码，观察登陆失败的页面
* 输入正确的用户名和密码，继续登陆
