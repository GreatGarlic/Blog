---
title: Spring 集成 Groovy
date: 2016-11-27 15:27:06
tags: [Misc, Util]
---

Groovy 是 用于 Java 虚拟机的一种敏捷的动态语言，很吸引人的一点是 Java 代码完全可以在 Groovy 里运行，Groovy 脚本可以动态的加载运行，例如网络游戏，为了游戏的平衡，每运行一段时间就会调整角色的攻击力计算公式(C, C++写的游戏一般会用 Lua 作为计算脚本)，如果是写在 Java 代码里，每次更新都要把主程序重新编译发布，代价是很大的，但是如果使用 Groovy 来提供计算，则只需要更新发布 Groovy 脚本(文本文件) 即可，估计也就是几 K。

<!--more-->

项目目录:

```
├── main
│   ├── java
│   │   ├── Main.java
│   │   └── com
│   │       └── xtuer
│   │           └── service
│   │               └── HelloService.java
│   └── resources
│       ├── groovy
│       │   └── com
│       │       └── xtuer
│       │           └── service
│       │               └── HelloServiceImpl.groovy
│       └── spring-beans.xml
└── test
    ├── java
    └── resources
```

## Spring 集成 Groovy 的步骤

1. 定义 Java 接口
2. Groovy 的类实现这个接口 (Groovy 脚本文件要放在资源文件里，否则编译的时候不会被自动复制到编译输出的文件夹中)
3. 在 Spring 的 Bean 配置文件里创建 Groovy 类的 bean
4. Java 代码里取得 Groovy 的 bean，然后使用

## Gradle 依赖

```groovy
compile 'org.springframework:spring-webmvc:4.3.0.RELEASE'
compile 'org.codehaus.groovy:groovy-all:2.4.5'
```

## 定义 Java 接口

```java
package com.xtuer.service;

public interface HelloService {
    String hello();
}
```

## Groovy 的类实现 Java 接口

```java
package com.xtuer.service;

public class HelloServiceImpl implements HelloService {
    String name;

    String hello() {
        return "Hello $name. Welcome to Groovy in Spring";
    }
}
```

## 在 spring-beans.xml 里创建 Groovy 类的 bean

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:lang="http://www.springframework.org/schema/lang"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/lang
       http://www.springframework.org/schema/lang/spring-lang.xsd">
    <lang:groovy id="helloService" script-source="classpath:groovy/com/xtuer/service/HelloServiceImpl.groovy">
        <lang:property name="name" value="Biao"/>
    </lang:groovy>
</beans>
```

* lang:groovy: 创建 Groovy 的 bean
* lang:property: Groovy 的 bean 也是可以设置属性的
* script-source: 指定 Groovy 脚本的路径
* refresh-check-delay="30000": 可以在 lang:groovy 里指定刷新脚本的时间属性

## Java 代码里取得 Groovy 的 bean，然后使用

```java
import com.xtuer.service.HelloService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Main {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("classpath:spring-beans.xml");
        HelloService service = context.getBean("helloService", HelloService.class);
        System.out.println(service.hello());
    }
}
```

输出:

```
Hello Biao. Welcome to Groovy in Spring
```

## 参考资料

[Groovy 在 Spring 中的简单使用](https://my.oschina.net/u/2494018/blog/610031)




