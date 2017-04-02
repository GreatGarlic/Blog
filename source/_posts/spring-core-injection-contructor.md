---
title: Contructor 注入
date: 2017-04-01 15:31:21
tags: Spring-Core
---
演示怎么给构造函数注入参数，使用指定的构造函数创建对象。

## Customer
```java
package com.xtuer.beans;

public class Customer {
    private String name;
    private int age;
    private String telephone;
    
    public Customer(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public Customer(String name, String telephone, int age) {
        this.name = name;
        this.telephone = telephone;
        this.age = age;
    }

    public Customer(String name, int age, String telephone) {
        this.name = name;
        this.telephone = telephone;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getTelephone() {
        return telephone;
    }

    public void setTelephone(String telephone) {
        this.telephone = telephone;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

<!--more-->

## spring-beans.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="customer" class="com.xtuer.beans.Customer">
        <constructor-arg value="Alice"/>
        <constructor-arg value="40"/>
        <constructor-arg value="1234567"/>
    </bean>
</beans>
```

## InjectionTest
```java
import com.xtuer.beans.Customer;
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
        Customer user = context.getBean("customer", Customer.class);
        CommonUtils.output(user);
    }
}
```

## 输出
```json
{
    "name" : "Alice",
    "age" : 1234567,
    "telephone" : "40"
}
```

> age 为什么是 1234567? 不是我们想要的！

## 构造函数的断定时有歧义
##### Customer 有 3 个构造函数
* public Customer(String name, int age)
* public Customer(String name, String telephone, int age)
* public Customer(String name, int age, String telephone)

上面 Bean 的定义使用有 3 个参数的构造函数来创建对象(因为有 3 个 `<constructor-arg>`)，找到了 2 个这样的构造函数，用了第一个语法上符合条件的构造函数来创建对象。但是在我们看来 40 应该是 age，最后 age 却是 1234567，与期望的不一样，也就是构造函数的断定上出现了歧义。如果构造函数有歧义的时候，可以指定构造函数的参数类型，用于确定要使用的构造函数：

```xml
<bean id="customer" class="com.xtuer.beans.Customer">
   <constructor-arg type="java.lang.String" value="Alice"/>
   <constructor-arg type="int" value="40"/>
   <constructor-arg type="java.lang.String" value="1234567"/>
</bean>
```

这次就能明确指定要使用的构造函数是 Customer(String name, int age, String telephone)

