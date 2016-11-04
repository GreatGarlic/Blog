---
title: 使用 Lombok 自动生成 Getter and Setter
date: 2016-10-13 13:25:52
tags: Java
---
根据 `Java Bean` 的规范，Bean 就是一个简单的类，主要是属性和访问函数 Getter and Setter 等，都是模版性的代码，虽然有 IDE 帮助我们自动生成，但是代码打开后全是一大堆的访问函数，看上去也不爽，更郁闷的是，有时候 Bean 有几十个属性，添加一个新的属性后也很容易忘了添加相应的访问函数，而 Java 的很多框架对属性的访问都是使用反射和访问函数来查找的，由于缺少访问函数导致有时候后端得到了数据，但是前端始终缺几个数据，逻辑看上去又没问题，很难得一下发现问题在哪里(已经发生过好几次)。现在好了，我们可以使用 `Lombok` 来自动的为 Bean 生成访问函数。

> Lombok 不会影响程序的性能，它使用 javac 的插件机制在编译阶段生成访问函数到 class 的字节码里，和我们直接写没有什么区别。

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
为了让 IDEA 对 Lombok 进行支持，为其装上 Lombok 插件就可以了(Eclipse 也有相应的插件)，否则在 IDEA 里编辑时提示找不到 Getter and Setter，不过编译运行仍然没问题，因为编译阶段会生成它们  
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

> 如果有 20，30，40 个属性，想象这个类的 Getter and Setter 会有多少，这个类最后会有多少代码。

## @Builder
Lombok 还可以使用 `Builder` 的模式来创建对象。

```java
import com.alibaba.fastjson.JSON;
import lombok.Builder;
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

    @Builder
    public User(int id, String username, String email) {
        this.id = id;
        this.username = username;
        this.email = email;
    }

    public static void main(String[] args) {
        User user = User.builder().id(6).username("Alice").email("alice@gmail.com").build();
        System.out.println(JSON.toJSONString(user)); // {"email":"alice@gmail.com","id":6,"username":"Alice"}
    }
}
```

> 如果 `@Builder` 作用在类名上，会自动的创建一个包含所有属性为参数的构造函数，这时如果想自己创建一个无参的构造函数，就必须同时自定义一个包含所有属性为参数的构造函数，而且参数的顺序必须和属性定义的顺序一致，否则会出问题，还是有点复杂的，为了简单起见，还是不要在类名上用 `@Builder`。
> 
> 如果 `@Builder` 只作用于构造函数上，那就相对容易了，参数的个数和顺序都是随意的。

## 参考资料
* Lombok 主页: <https://projectlombok.org>
