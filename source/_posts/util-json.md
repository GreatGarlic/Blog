---
title: Jackson 处理 JSON
date: 2016-04-24 14:10:49
tags: [Java, Util]
---

使用 Jackson 序列化和反序列化对象，主要使用 `ObjectMapper` 的 2 个方法: `writeValueAsString()` 和 `readValue()`。

<!--more-->

## Gradle 依赖
```groovy
dependencies {
    compile 'com.fasterxml.jackson.core:jackson-databind:2.7.3'
}
```

## JsonUtil
```java
import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.io.IOException;

public class JsonUtil {
    private static ObjectMapper mapper = new ObjectMapper();

    static {
        mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL); // 不序列化 null
    }

    public static String toJson(Object obj) {
        try {
            return mapper.writeValueAsString(obj);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }

        return null;
    }

    public static <T> T fromJson(String json, Class<T> clazz) {
        try {
            return mapper.readValue(json, clazz);
        } catch (IOException e) {
            e.printStackTrace();
        }

        return null;
    }

    public static void main(String[] args) {
        User user = new User(1, "Alice", "alice@gmail.com");

        String json = JsonUtil.toJson(user);
        System.out.println(json);

        user = JsonUtil.fromJson(json, User.class);
        System.out.printf("id: %d, username: %s, email: %s", user.getId(), user.getUsername(), user.getEmail());
    }
}
```

输出

```
{"id":1,"username":"Alice","email":"alice@gmail.com"}
id: 1, username: Alice, email: alice@gmail.com
```

## User
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

    // Getters and setters
}
```
