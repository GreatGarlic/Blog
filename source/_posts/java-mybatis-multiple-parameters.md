---
title: MyBatis 传递多个参数
date: 2018-02-27 13:41:37
tags: Java
---

MyBatis 传递多个参数一般有以下几种方法:

* 使用 Map
* 把参数封装成 Bean，传递 Bean 的对象
* 使用 `@Param`
* 编译时使用 `-parameters` 参数 (推荐使用)

下面以用户名和密码作为参数查询用户为例进行介绍。<!--more-->

## 使用 Map

Java Mapper:

```java
public interface UserMapper {
    User findUserByUsernameAndPassword(Map params);
}
```

Xml Mapper:

```xml
<select id="findUserByUsernameAndPassword" resultType="User">
    SELECT id, username, password FROM user WHERE username=#{username} AND password=#{password}
</select>
```

调用:

```java
Map params = new HashMap();
params.put("username", username);
params.put("password", password);
User user = userMapper.findUserByUsernameAndPassword(params);
```

缺点:

每次都要构造 Map，不方便。

## 把参数封装成 Bean，传递 Bean 的对象

Java Mapper:

```java
public interface UserMapper {
    User findUserByUsernameAndPassword(UserParam param);
}
```

UserParam:

```java
public class UserParam {
    private String username;
    private String password;
    ...// Getters and Setters
}
```

Xml Mapper:

```xml
<select id="findUserByUsernameAndPassword" resultType="User">
    SELECT id, username, password FROM user WHERE username=#{username} AND password=#{password}
</select>
```

调用:

```java
UserParam param = new UserParam();
param.setUsername(username);
param.setPassword(password);
User user = userMapper.findUserByUsernameAndPassword(param);
```

缺点:

一种参数组合就要创建一个对应的类，麻烦。

## 使用 @Param

Java Mapper:

```java
public interface UserMapper {
    User findUserByUsernameAndPassword(@Param("username") String username, @Param("password") String password);
}
```

Xml Mapper:

```xml
<select id="findUserByUsernameAndPassword" resultType="User">
    SELECT id, username, password FROM user WHERE username=#{username} AND password=#{password}
</select>
```

调用:

```java
User user = userMapper.findUserByUsernameAndPassword(username, password);
```

缺点:

有的时候参数列表会写的很长。

## 编译时使用 `-parameters` 参数

使用 javac 编译时增加参数 `-parameters` 就能够把参数的名字编译到字节码里，MyBatis 就能够使用参数的名字进行参数绑定，不需要使用 `@Param` 了，代码看上去简洁很多，不过需要 JDK 最低 Java 8 才行。

> javac -parameters: Generate metadata for reflection on method parameters.

Java Mapper:

```java
public interface UserMapper {
    User findUserByUsernameAndPassword(String username, String password);
}
```

Xml Mapper:

```xml
<select id="findUserByUsernameAndPassword" resultType="User">
    SELECT id, username, password FROM user WHERE username=#{username} AND password=#{password}
</select>
```

调用:

```java
User user = userMapper.findUserByUsernameAndPassword(username, password);
```

缺点:

需要最低 Java 8，并且在编译时使用 `-parameters` 参数，但是代码简洁。

---

### Gradle 在任务 compileJava 中增加编译参数:

```groovy
compileJava {
    options.compilerArgs << '-Xlint:unchecked' << '-Xlint:deprecation' << '-parameters'
}
```

### IDEA 中设定额外的编译参数:

在 `Compiler -> Java Compiler` 中，设置 `Additional command line parameters` 为 `-parameters`:

![](/img/java/idea-javac-parameters.png)