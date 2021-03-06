---
title: Setter 注入
date: 2017-04-01 15:26:35
tags: SpringCore
---
## CommonUtils

```java
package com.xtuer.util;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.core.util.DefaultIndenter;
import com.fasterxml.jackson.core.util.DefaultPrettyPrinter;
import com.fasterxml.jackson.databind.ObjectMapper;

public class CommonUtils {
    private static DefaultPrettyPrinter printer;
    static {
        // Setup a pretty printer with an indenter (indenter has 4 spaces in this case)
        DefaultPrettyPrinter.Indenter indenter = new DefaultIndenter("    ", DefaultIndenter.SYS_LF);
        printer = new DefaultPrettyPrinter();
        printer.indentObjectsWith(indenter);
        printer.indentArraysWith(indenter);
    }

    /**
     * 以 Json 的格式输出对象
     * @param obj
     */
    public static void output(Object obj) {
        ObjectMapper mapper = new ObjectMapper();

        try {
            // 默认用 2 个空格缩进
            // System.out.println(mapper.writerWithDefaultPrettyPrinter().writeValueAsString(obj));

            // 自定义缩进，用 4 个空格
            System.out.println(mapper.writer(printer).writeValueAsString(obj));
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
    }
}
```

<!--more-->

## User

```java
package com.xtuer.beans;

public class User {
    private int id;
    private String username;
    private String password;
    private Address address;
    
    // 省略 setter and getter
}
```

## Address

```java
package com.xtuer.beans;

public class Address {
    private int id;
    private String country;
    private String province;
    private String street;
    
    // 省略 setter and getter
}
```

## spring-beans.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="address" class="com.xtuer.beans.Address">
        <property name="country" value="China"/>
        <property name="province" value="BeiJing"/>
        <property name="street" value="ChaoYang"/>
    </bean>

    <bean id="user" class="com.xtuer.beans.User">
        <property name="username" value="Alice"/>
        <property name="password" value="Passw0rd"/>
        <property name="address" ref="address"/>
    </bean>
</beans>
```

## InjectionTest
```java
import com.xtuer.beans.User;
import com.xtuer.util.CommonUtils;
import org.junit.BeforeClass;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class InjectionTest {
    private static ApplicationContext context;

    @BeforeClass
    public static void setup() {
        context = new ClassPathXmlApplicationContext("spring-beans.xml");
    }

    @Test
    public void test() {
        User user = context.getBean("user", User.class);
        CommonUtils.output(user);
    }
}
```

## 输出
```js
{
    "id" : 0,
    "username" : "Alice",
    "password" : "Passw0rd",
    "address" : {
        "id" : 0,
        "country" : "China",
        "province" : "BeiJing",
        "street" : "ChaoYang"
    }
}
```

## 说明
1. `<property name="username" value="Alice"/>` 调用 user.setUsername("Alice") 设置属性 username。
2. `<property name="address" ref="address"/>`，找到标志为 **address**(可以是 id, name, alias) 的 Bean，然后调用 user.setAddress(address) 设置属性 address。

## 注入方式: attribute 和 element
* value 和 ref 都可以使用 attribute 和 element 的方式注入，推荐尽量用 attribute 的方式，这样会简洁一些
* value 用于简单类型: primitive type(int, boolean, float 等), String, Date, 还有注册了 PropertyEditor 的类型(把字符串转换成对象)。
* ref 用于引用 Bean Configuration File 里定义的另一个 Bean


## value 用 element 的方式如下注入:
```xml
<bean id="user" class="com.xtuer.beans.User">
  <property name="username">
      <value>Alice</value>
  </property>
  <property name="password" value="Passw0rd"/>
  <property name="address" ref="address"/>
</bean>
```
## ref 用 element 的方式如下注入:
```xml
<bean id="user" class="com.xtuer.beans.User">
  <property name="username" value="Alice"/>
  <property name="password" value="Passw0rd"/>
  <property name="address">
      <ref bean="address"/>
  </property>
</bean>
```

## 注入匿名对象
每创建一次 User，都会新创建一个 Address 的匿名对象。
使用 `<bean>`，不能设置 id, name 等。

```xml
<bean id="user" class="com.xtuer.beans.User">
   <property name="username" value="Alice"/>
   <property name="password" value="Passw0rd"/>
   <property name="address">
       <bean class="com.xtuer.beans.Address">
           <property name="country" value="Germany"/>
           <property name="province" value="Braunschweig"/>
           <property name="street" value="Wiesenstrasse"/>
       </bean>
   </property>
</bean>
```
