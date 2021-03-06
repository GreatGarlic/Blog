---
title: Enum 注入
date: 2017-09-22 11:49:31
tags: SpringCore
---

注入 Enum 可以使用字符串直接注入，或者借助 org.springframework.beans.factory.config.FieldRetrievingFactoryBean，下面以注入 FastJson 的 SerializerFeature 和自定义 enum 为例. 

```java
package com.alibaba.fastjson.serializer;

public enum SerializerFeature {
    QuoteFieldNames,
    UseSingleQuotes,
    WriteMapNullValue,
    WriteEnumUsingToString,
    WriteEnumUsingName,
    ...
}
```

```java
public enum Color {
    RED, GREEN, BLUE
}
```

<!--more-->

## enum.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="quoteFieldNames" class="org.springframework.beans.factory.config.FieldRetrievingFactoryBean">
        <property name="staticField" value="com.alibaba.fastjson.serializer.SerializerFeature.QuoteFieldNames" />
    </bean>

    <bean id="injectEnum" class="InjectEnum">
        <property name="color" value="RED"/>
        <property name="features">
            <list> <!--表示是 list-->
                <ref bean="quoteFieldNames"/>
                <value>UseSingleQuotes</value>
            </list>
        </property>
    </bean>
</beans>
```

## InjectEnum.java

```java
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.serializer.SerializerFeature;
import lombok.Getter;
import lombok.Setter;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

@Getter
@Setter
public class InjectEnum {
    private SerializerFeature features[];
    private Color color;

    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("enum.xml");
        InjectEnum obj = context.getBean("injectEnum", InjectEnum.class);

        System.out.println(JSON.toJSONString(obj));
    }
}
```

输出:

```
{"color":"RED","features":["QuoteFieldNames","UseSingleQuotes"]}
```

