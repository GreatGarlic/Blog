---
title: Spring 数据绑定之类型转换
date: 2016-04-18 23:03:04
tags: SpringMVC
---

Spring 里常用的类型转换工具有

* PropertyEditor: 字符串转换为对象, Controller 里使用
* Formatter: 字符串和对象互相转换, spring-mvc 配置里使用
* Converter: 不同类型之间互相转换, spring-mvc 配置里使用

<!--more-->

## UserFormatter
```java
package com.xtuer.formatter;

import com.xtuer.bean.User;
import org.springframework.format.Formatter;

import java.text.ParseException;
import java.util.Locale;

// Formatter 是全局的: 字符串和对象互相转换
public class UserFormatter implements Formatter<User> {
    // "Bob, 123"
    @Override
    public User parse(String text, Locale locale) throws ParseException {
        String[] components = text.split(",");
        User user = new User();
        user.setUsername(components[0].trim());
        user.setAge(Integer.parseInt(components[1].trim()));

        return user;
    }

    @Override
    public String print(User object, Locale locale) {
        return null;
    }
}
```

## AdminConverter
```java
package com.xtuer.converter;

import com.xtuer.bean.Admin;
import org.springframework.core.convert.converter.Converter;

public class AdminConverter implements Converter<String, Admin> {
    // "Bob, 123"
    @Override
    public Admin convert(String source) {
        String[] components = source.split(",");
        Admin admin = new Admin();
        admin.setUsername(components[0].trim());
        admin.setAge(Integer.parseInt(components[1].trim()));

        return admin;
    }
}
```

## 注册 Formatter 和 Converter 到 Spring 里
```xml
<mvc:annotation-driven conversion-service="customConversionService"/>

<bean id="customConversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
    <property name="formatters">
        <bean class="com.xtuer.formatter.UserFormatter"/>
    </property>
    <property name="converters">
        <bean class="com.xtuer.converter.AdminConverter"/>
    </property>
</bean>
```

## Controller
```java
    // http://localhost:8080/format?user=Alice,123
    @RequestMapping("/format")
    @ResponseBody
    public User format(User user) {
        return user;
    }

    // http://localhost:8080/convert?admin=Bob,123
    @RequestMapping("/convert")
    @ResponseBody
    public Admin convert(Admin admin) {
        return admin;
    }
```

> 如果只是字符串和对象之间转换，Formatter 和 Converter 的功能是一样的，SpringMVC 的数据绑定也只是把 URL 中的参数部分转换为对象，所以这时使用 Formatter 或者 Converter 都是一样的。如果从语意上来说，Formatter 用于格式化输出时好一些，可以有本地化的信息，例如日期，货币，时间等，而 Converter 用于类型转换更恰当。

## DateEditor
```java
    // http://localhost:8080/editor?date=2016-04-18
    @RequestMapping("/editor")
    @ResponseBody
    public Date editor(Date date) {
        return date;
    }
    
    @InitBinder("date")
    public void initDate(WebDataBinder binder) {
        binder.registerCustomEditor(Date.class, new CustomDateEditor(new SimpleDateFormat("yyyy-MM-dd"), true));
    }

// 下面自定义的 UserEditor 不生效(没和 UserFormatter 一起使用)
//    @InitBinder("user")
//    public void initUserWithEditor(WebDataBinder binder) {
//        binder.registerCustomEditor(User.class, new UserEditor());
//    }
```

> 比较诡异的问题是自己实现一个 Editor 继承自 PropertyEditorSupport，然后使用 `@InitBinder` 注册为 custom editor，不会生效，但是上面注册的 CustomDateEditor 却能生效。
> 自定义 PropertyEditor 还有一个很重要的用途就是在 Spring Bean 配置文件里配置好后定义 Bean 时可以把字符串转换为 Bean 的属性，例如下面的例子

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--注册自定义的 PropertyEditor-->
    <bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">
        <property name="customEditors">
            <map>
                <!--Key 是要生成对象的类名，Value 是 Editor 的类名-->
                <entry key="com.xtuer.beans.Address" value="com.xtuer.beans.editor.AddressEditor"/>
            </map>
        </property>
    </bean>

    <bean id="user" class="com.xtuer.beans.User">
        <property name="username" value="Alice"/>
        <property name="password" value="Passw0rd"/>
        
        <!--Spring 会把字符串 "2|China|BeiJing|WangJin" 自动的转换为 Address 的对象-->
        <property name="address" value="2|China|BeiJing|WangJin"/>
    </bean>
</beans>
```

> Formatter 和 Converter 不能用于 Spring Bean 配置文件里把字符串转换为对象，只能用于 SpringMVC 的 Controller 里把 URL 中的字符串转换为对象，而 PropertyEditor 2 个地方都可以使用。

## User
```java
public class User {
    private String username;
    private int age;
}
```

## Admin
```java
public class Admin {
    private String username;
    private int age;
}
```

