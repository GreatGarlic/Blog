---
title: Spring Boot Thymeleaf
date: 2017-09-11 22:35:42
tags: SpringBoot
---

SpringBoot 默认使用 Thymeleaf 2，为了使用 Thymeleaf 3，需要引入下面的依赖:

```groovy
compile('org.springframework.boot:spring-boot-starter-thymeleaf')
compile('org.thymeleaf:thymeleaf:3.0.7.RELEASE')
compile('org.thymeleaf:thymeleaf-spring4:3.0.7.RELEASE')
compile('nz.net.ultraq.thymeleaf:thymeleaf-layout-dialect:2.2.2')
```

application.properties 中配置:

```ini
spring.thymeleaf.mode=HTML5
spring.thymeleaf.cache=false
spring.thymeleaf.suffix=.html
spring.thymeleaf.encoding=UTF-8
spring.thymeleaf.content-type=text/html
```

Controller:

```java
@GetMapping("/hello")
public String hello() {
    return "hello"; // View 的名字为 hello.html
}
```

hello.html:

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8"/>
</head>

<body>
    Thymeleaf
</body>

</html>
```

