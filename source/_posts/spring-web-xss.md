---
title: 防止 XSS 攻击
date: 2016-10-18 11:31:23
tags: Spring-Web
---
可以使用 `XSS Filter` 防止 XSS 攻击，具体细节请访问 <http://www.servletsuite.com/servlets/xssflt.htm>

使用步骤:

1. 把 `xssflt.jar` 放到 `WEB-INF/lib`
2. 把 `xssflt.jar` 添加到 Gradle 依赖

    ```groovy
    compile fileTree(dir: 'src/main/webapp/WEB-INF/lib', include: ['*.jar'])
    ```
3. 修改 web.xml

    ```xml
    <!-- 防止 XSS 攻击 -->
    <filter>
        <filter-name>XSSFilter</filter-name>
        <filter-class>com.cj.xss.XSSFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>XSSFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ```

<!--more-->

## 表单页面

```html
<!DOCTYPE html>
<html>
<head>
    <title>XSS Test</title>
    <meta charset="UTF-8">
</head>
<body>
    <form class="" action="/a" method="post">
        <input type="text" name="name" value="">
        <button type="submit" name="button">提交</button>
    </form>
</body>
</html>
```

## 服务器端

```java
@PostMapping("/a")
@ResponseBody
public String postA(@RequestParam String name) {
    System.out.println(name);
    return name;
}
```

## 测试一
1. 输入 `test`
2. 控制台输出 `test`
3. 浏览器端收到 `test` 

## 测试二
1. 输入 `<test>`
4. 控制台输出 `&lt;test&gt;`
5. 浏览器端收到 `&lt;test&gt;`
6. 说明防止 XSS 攻击生效
