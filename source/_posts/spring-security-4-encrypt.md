---
title: Spring Security 加密密码
date: 2016-04-10 10:45:54
tags: SpringSecurity
---

明文保存密码是不可取的，可以使用 `SHA`，`BCrypt` 等对密码进行加密。

BCrypt 算法与 MD5/SHA 算法有一个很大的区别，每次生成的 hash 值都是不同的，就可以免除存储 salt，暴力破解起来也更困难。BCrypt 加密后的字符长度比较长，有60位，所以用户表中密码字段的长度，如果打算采用 BCrypt 加密存储，字段长度不得低于 68(需要前缀 `{bcrypt}`)。

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

随意取其中任意一个都可以，因为每次生成都是不一样的，所以取第一个就可以了。<!--more-->

## Spring Security 使用 BCrypt 加密
Spring Security 4 的时候需要配置 `<password-encoder hash="bcrypt"/>` 指定加密方式，Spring Security 5 不需要配置了，而是用户密码加前缀的方式表明加密方式，例如

* `{bcrypt}$2a$10$gQKUOoFuevnCkoej3.AvAO9YzHKCKYmKuiSfEGHL22piY2FfNDQYu` 说明是使用 BCrypt 进行加密的

* `{noop}Passw0rd` 则是使用明文保存的密码 (noop: No Operation)


这样的好处是同一个系统可以使用多种加密方式，迁移用户到新系统时比较就省事了。Spring Security 5 默认支持的密码加密方式在 PasswordEncoderFactories 中定义:

```java
public static PasswordEncoder createDelegatingPasswordEncoder() {
    String encodingId = "bcrypt";
    Map<String, PasswordEncoder> encoders = new HashMap<>();
    encoders.put(encodingId, new BCryptPasswordEncoder());
    encoders.put("ldap", new LdapShaPasswordEncoder());
    encoders.put("MD4", new Md4PasswordEncoder());
    encoders.put("MD5", new MessageDigestPasswordEncoder("MD5"));
    encoders.put("noop", NoOpPasswordEncoder.getInstance());
    encoders.put("pbkdf2", new Pbkdf2PasswordEncoder());
    encoders.put("scrypt", new SCryptPasswordEncoder());
    encoders.put("SHA-1", new MessageDigestPasswordEncoder("SHA-1"));
    encoders.put("SHA-256", new MessageDigestPasswordEncoder("SHA-256"));
    encoders.put("sha256", new StandardPasswordEncoder());

    return new DelegatingPasswordEncoder(encodingId, encoders);
}
```

## UserService

> 从数据源取得的密码是加密后的密码

```java
public class UserService {
    private static Map<String, User> users = new HashMap<String, User>();

    static {
        // 模拟数据源，可以是多种，如数据库，LDAP，从配置文件读取等
        users.put("admin", new User("admin", "{noop}Passw0rd", "ROLE_ADMIN")); // 密码是 Passw0rd
        users.put("alice", new User("alice", "{bcrypt}$2a$10$dtA5fPvVJEBHLPp7FZci9uKJL90zF8T1EQZzP9qownQlf130bdBZW", "ROLE_USER")); // 密码是 Passw0rd
    }

    public User findUserByUsername(String username) {
        return users.get(username);
    }
}
```

## 测试

* 访问 http://localhost:8080/admin
* 输入错误的用户名或密码，观察登陆失败的页面
* 输入正确的用户名和密码，继续登陆
