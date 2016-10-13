---
title: 使用 Lombok 自动生成 Getter and Setter
date: 2016-10-13 13:25:52
tags: Java
---
根据 `Java Bean` 的规范，Bean 就是一个简单的类，主要是属性和访问函数 Getter and Setter 等，都是模版性的代码，虽然有 IDE 帮助我们自动生成，但是代码打开后全是一大堆的访问函数，看上去也不爽，现在好了，我们可以使用 `Lombok` 来自动的为 Bean 生成访问函数。

<!--more-->

## Gradle 依赖
```groovy
compile 'org.projectlombok:lombok:1.16.10'
```

## 使用 Lombok 的 Bean
* 只需要在类名前加上 `@Getter`, `@Setter` 即可(Lombok 还有其他几个注解，不过没必要使用，影响代码的可读性)。
* 如果我们提供了属性的访问函数，则 Lombok 不会为其再生成。

```java
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class User {
    private int id;
    private String username;
    private String email;

    public User() {
    }

    public User(int id, String username, String email) {
        this.id = id;
        this.username = username;
        this.email = email;
    }

    // 如果提供了访问函数，则 Lombok 不会为其再生成
    public String getEmail() {
        return "Email: " + email;
    }
}
```

## 测试
```java
public class Test {
    public static void main(String[] args) {
        User user = new User(1, "Alice", "alice@gmail.com");
        System.out.println(user.getUsername());
        System.out.println(user.getEmail());
    }
}
```

输出:

> Alice  
> Email: alice@gmail.com

## Lombok 插件
为了让 IDEA 对 Lombok 进行支持，为其装上 Lombok 插件就可以了(Eclipse 也有相应的插件)  
![](/img/java/lombok-idea.png)

## 不用 Lombok 的 Bean
```java
public class User {
    private int id;
    private String username;
    private String email;

    public User() {
    }

    public User(int id, String username, String email) {
        this.id = id;
        this.username = username;
        this.email = email;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
```

## 参考资料
* Lombok 主页: <https://projectlombok.org>
