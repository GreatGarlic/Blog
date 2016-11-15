---
title: SpringMVC 数据绑定
date: 2016-04-17 08:57:33
tags: Spring-Mvc
---

SpringMVC 中提供了多种数据绑定，可以把请求中的数据绑定为简单类型，简单数组，对象，对象的数组等。

<!--more-->

## 简单数组
* <http://localhost:8080/array?name=Tom&name=Lucy>
* URL 中多个同名的参数映射为同一个数组的元素，例如 `name`
* 数组是 primitive, String 等简单类型的数组

```java
    @RequestMapping("/array")
    @ResponseBody
    public String[] array(@RequestParam("name") String[] names) {
        return names;
    }
```

```java
    @RequestMapping("/list")
    @ResponseBody
    public List<String> list(@RequestParam("name") List<String> names) {
        return names;
    }
```

显示:

> ["Tom","Lucy"]

## 简单对象

* <http://localhost:8080/object?username=Tom&age=10>
* URL 中参数名和对象中的属性名一样的会自动映射
* 对 `User` 进行映射

```java
    @RequestMapping("/object")
    @ResponseBody
    public User object(User user) {
        return user;
    }
```

显示:

```js
{
    "username": "Tom",
    "age": 10,
    "address": null
}
```

## 复杂对象
* <http://localhost:8080/object?username=Tom&password=Passw0rd&address.city=Beijing&address.street=SCI>
* 可以使用级连映射，URL 的参数名为属性的级连，例如 address.city，映射的是 user 的 address 的 city 属性

```java
    @RequestMapping("/nested-object")
    @ResponseBody
    public User nestedObject(User user) {
        return user;
    }
```

显示:

```js
{
    "username": "Tom",
    "age": 10,
    "address": {
        "city": "Beijing",
        "street": "SCI"
    }
}
```

## 同属性多对象
* <http://localhost:8080/intersect-object?user.username=Tom&admin.username=Jim&age=10>
* User 和 Admin 都有同名的属性 username, age，可以使用 `WebDataBinder` 对其区分
* 映射的参数名 `user` 和 @InitBinder 中的 value `user` 要一样

> 如果 URL 都使用 username 和 age 的话，会同时作用于 user 和 admin，使用 `WebDataBinder` 可对其加前缀，就能在映射的时候区分开是给谁使用的。  
> 
> `user.username` `and admin.username` 分别映射到对象 user 和 admin，age 同时映射到他们的 age 属性上。

```java
    @RequestMapping("/intersect-object")
    @ResponseBody
    public Map intersectObject(User user, Admin admin) {
        Map<String, Object> map = new HashMap<String, Object>();
        map.put("user", user);
        map.put("admin", admin);
        return map;
    }

    @InitBinder("user")
    public void initUser(WebDataBinder binder) {
        binder.setFieldDefaultPrefix("user.");
    }

    @InitBinder("admin")
    public void initAdmin(WebDataBinder binder) {
        binder.setFieldDefaultPrefix("admin.");
    }
```

显示:

```js
{
    "admin": {
        "username": "Jim",
        "age": 10
    },
    "user": {
        "username": "Tom",
        "age": 10,
        "address": null
    }
}
```

## 对象数组
* <http://localhost:8080/object-list?users[0].username=Tom&users[1].username=Lucy>
* 不能直接使用 `List<User> users` 来映射
* 需要创建一个类 UserList，其属性为 `List<User> users`，然后使用级连映射相似的方式来映射，但是要有数组的下标

```java
    @RequestMapping("/object-list")
    @ResponseBody
    public UserList objectList(UserList userList) {
        return userList;
    }
```

显示:

```js
{
    "users": [{
        "username": "Tom",
        "age": 0,
        "address": null
    }, {
        "username": "Lucy",
        "age": 0,
        "address": null
    }]
}
```

## 对象的 Map
* <http://localhost:8080/object-map?users['x'].username=Tom&users['y'].username=Lucy>
* <http://localhost:8080/object-map?users["x"].username=Tom&users["y"].username=Lucy>
*  不能直接使用 `Map<String, User> users` 来映射
*  需要创建一个类 UserMap，其属性为 `Map<String, User> users`，然后使用级连映射相似的方式来映射，但是要有 Map 的 key

```java
    @RequestMapping("/object-map")
    @ResponseBody
    public UserMap objectMap(UserMap users) {
        return users;
    }
```

显示:

```js
{
    "users": {
        "x": {
            "username": "Tom",
            "age": 0,
            "address": null
        },
        "y": {
            "username": "Lucy",
            "age": 0,
            "address": null
        }
    }
}
```

## AJAX 数据绑定
> 如果是 AJAX 传递数据给服务器端，然后要绑定为对象，那么 AJAX 不能是 GET 操作，如果只是使用 AJAX 从服务器端获取数据，则可以是 GET，POST 等。

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <script src="http://apps.bdimg.com/libs/jquery/2.1.1/jquery.min.js"></script>
    <script>
    $(document).ready(function() {
        var data = {username: 'Biao', age: 111};

        $.ajax({
            url: '/json-bind',
            type: 'POST', // 1. 不能是 GET
            dataType: 'json',
            contentType: 'application/json', // 2. 少了就会报错
            data: JSON.stringify(data) // 3. data 需要序列化一下
        })
        .done(function(result) {
            console.log(result);
        })
        .fail(function() {
            console.log("error");
        })
        .always(function() {
            console.log("complete");
        });
    });
    </script>
</head>
<body>
</body>
</html>
```

> 接受 AJAX 传递过来的数据，映射为对象时必须要有 `@RequestBody` 修饰方法的参数

```java
    @RequestMapping("/json-bind")
    @ResponseBody
    public User json(@RequestBody User user) {
        System.out.println(user.getUsername());
        return user;
    }
```

浏览器控制台输出

```js
{
    username: "Biao",
    age: 111,
    address: null
}
```

## 需要的类
### User
```java
public class User {
    private String username;
    private int age;
    private Address address;

    // Setters and getters
}
```

### Address
```java
public class Address {
    private String city;
    private String street;

    // Setters and getters
}
```

### Admin
```java
public class Admin {
    private String username;
    private int age;

    // Setters and getters
}
```

### UserList
```java
public class UserList {
    private List<User> users;

    // Setters and getters
}
```

### UserMap
```java
public class UserMap {
    private Map<String, User> users;

    // Setters and getters
}
```
