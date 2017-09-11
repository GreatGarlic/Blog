---
title: SpringBoot Start
date: 2017-09-05 13:51:36
tags: SpringBoot
---

Spring Boot 创建入门级 RESTful Web 项目简单到令人发指，下面就来看看怎么用吧(如果想知道 Spring Boot 是啥，搜索即可):

1. 创建项目的骨架
2. 添加 RestController
3. 启动项目: gradle bootRun 
4. 打包项目: gradle build 

## 创建项目的骨架

1. 访问 <http://start.spring.io>
2. 选择 Gradle: `Generate a Gradle Project with Java Spring Boot 1.5.6`
3. 填写 Group(项目的包名，例如 com.xtuer) 和 Artifact(可不填)
4. Search for dependencies 输入 **web**
5. 点击 Generate Project，会自动下载项目骨架的 zip 文件
6. 解压，如果不需要里面的 gradlew，删除即可<!--more-->

## 添加 RestController

```java
package com.xtuer;

import org.springframework.web.bind.annotation.*;

@RestController
public class XController {
    @GetMapping("/hello")
    @ResponseBody
    public String hello() {
        return "Hello";
    }
}
```

## 启动项目

终端进入项目目录，执行 `gradle bootRun`，输出日志可以看到项目启动成功，然后浏览器访问 <http://localhost:8080/hello>，得到响应的字符串 **Hello**，完成.

```
Starting a Gradle Daemon (subsequent builds will be faster)

> Configure project :
Repository https://repo1.maven.org/maven2/ replaced by http://maven.aliyun.com/nexus/content/groups/public/.

> Task :bootRun

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.6.RELEASE)

2017-09-05 14:00:29.168  INFO 30586 --- [           main] com.xtuer.Application                    : Starting Application on Biao.local with PID 30586 (/Users/Biao/Desktop/demo/build/classes/java/main started by Biao in /Users/Biao/Desktop/demo)
2017-09-05 14:00:29.171  INFO 30586 --- [           main] com.xtuer.Application                    : No active profile set, falling back to default profiles: default
2017-09-05 14:00:29.218  INFO 30586 --- [           main] ationConfigEmbeddedWebApplicationContext : Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@27fe3806: startup date [Tue Sep 05 14:00:29 CST 2017]; root of context hierarchy
2017-09-05 14:00:30.039  INFO 30586 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat initialized with port(s): 8080 (http)
2017-09-05 14:00:30.051  INFO 30586 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
...
2017-09-05 14:00:30.459  INFO 30586 --- [           main] s.w.s.m.m.a.RequestMappingHandlerAdapter : Looking for @ControllerAdvice: org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@27fe3806: startup date [Tue Sep 05 14:00:29 CST 2017]; root of context hierarchy
2017-09-05 14:00:30.515  INFO 30586 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/hello],methods=[GET]}" onto public java.lang.String com.xtuer.XController.hello()
...
2017-09-05 14:00:30.733  INFO 30586 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
2017-09-05 14:00:30.736  INFO 30586 --- [           main] com.xtuer.Application                    : Started Application in 11.847 seconds (JVM running for 12.133)
...
```

## 打包项目

终端进入项目目录，执行 `gradle build`，在 `<project>/build/libs` 中得到 jar 包，可以使用 `java -jar xxx.jar` 执行.