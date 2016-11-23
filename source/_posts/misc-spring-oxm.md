---
title: Spring O/X Mapping
date: 2016-11-22 18:23:47
tags: [Misc, Spring-Core]
---
Spring 3.0 的一个新特性是 `O/X Mapper`，下面简称 `OXM`，O 代表 Object，X 代表 XML，它的目的是实现 Java POJO 对象和 XML 文档之间的互相转换。

Spring O/X Mapper 定义统一的接口，实现由第三方框架提供。要使用 Spring 的 O/X 功能，您需要一个在 Java 对象和 XML 之间来回转换的库，Castor 就是这样一个流行的第三方工具，本文将使用这个工具，其他这样的工具包括 XMLBeans、Java Architecture for XML Binding (JAXB)、JiBX 和 XStream。

<!--more-->

项目目录:

```
├── main
│   ├── java
│   │   └── com
│   │       └── xtuer
│   │           └── bean
│   │               └── User.java
│   └── resources
│       ├── castor.properties
│       ├── oxm-user.xml
│       └── spring-beans.xml
└── test
    ├── java
    │   └── TestOxm.java
    └── resources
```

* User 是一个普通的 Java 类，即 POJO
* spring-beans.xml 为 Spring 的配置文件
* oxm-user.xml 用来描述类的属性映射到 xml 时的 tag name，OXM 的好处是对象的属性名和 xml 里面对应的 tag name 可以不一样，这样在不同的系统间交互数据时很有用
* TestOxm 为测试类
* castor.properties: 默认生成的 xml 是没有缩进的，不利于阅读，为了生成格式化的 xml，需要在配置文件 castor.properties 中定义 `org.exolab.castor.indent = true`，Castor 会读取这个文件

> **术语:**
> 
> * Marshalling: 把 POJO 映射为 xml
> * Unmarshalling: 把 xml 映射为 POJO

## Gradle 依赖
```groovy
compile 'org.springframework:spring-webmvc:4.3.0.RELEASE'
compile 'org.springframework:spring-oxm:4.3.0.RELEASE'
compile 'org.codehaus.castor:castor-xml:1.4.1'

// 下面几个依赖都是辅助编码用的，oxm 只需要上面 3 个依赖
compile 'commons-io:commons-io:2.4'
compile 'org.projectlombok:lombok:1.16.10'
compile 'com.alibaba:fastjson:1.2.17'
    
testCompile 'org.springframework:spring-test:4.3.0.RELEASE'
testCompile 'junit:junit:4.12'
```

## User.java
```java
package com.xtuer.bean;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class User {
    private int id;
    private String username;
    private String password;

    public User() {
    }

    public User(int id, String username, String password) {
        this.id = id;
        this.username = username;
        this.password = password;
    }
}
```

## oxm-user.xml
```xml
<mapping>
    <class name="com.xtuer.bean.User">
        <map-to xml="user"/>

        <field name="id" type="integer">
            <bind-xml name="user_id" node="element"/>
        </field>
        <field name="username" type="string">
            <bind-xml name="username" node="element"/>
        </field>
        <field name="password" type="string">
            <bind-xml name="password" node="element"/>
        </field>
    </class>
</mapping>
```

> `map-to` 的 `xml` 的属性值是生成的 xml 的根节点的标签名
> `field` 的 `name` 的属性值是 POJO 的属性名  
> `bind-xml` 的 `name` 的属性值是生成的 xml 中属性对于的标签名

## spring-beans.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="marshaller" class="org.springframework.oxm.castor.CastorMarshaller">
        <property name="mappingLocation" value="classpath:oxm-user.xml" />
    </bean>
</beans>
```

> CastorMarshaller 同时实现了 Marshaller 和 Unmarshaller 接口

## castor.properties
```
org.exolab.castor.indent = true
```

## 测试: TestOxm.java
```java
import com.alibaba.fastjson.JSON;
import com.xtuer.bean.User;
import org.apache.commons.io.FileUtils;
import org.apache.commons.io.IOUtils;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.oxm.castor.CastorMarshaller;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringRunner;

import javax.xml.transform.stream.StreamResult;
import javax.xml.transform.stream.StreamSource;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;

@RunWith(SpringRunner.class)
@ContextConfiguration({"classpath:spring-beans.xml"})
public class TestOxm {
    @Autowired
    private CastorMarshaller marshaller;

    private static final String FILE_NAME = "/Users/Biao/Desktop/user.xml";

    /**
     * 把 User 的对象 marshalling 生成 xml 文件 user.xml
     */
    @Test
    public void testMarshalling() throws Exception {
        User user = new User(12, "Alice", "Passw0rd");
        FileOutputStream fos = FileUtils.openOutputStream(new File(FILE_NAME));
        marshaller.marshal(user, new StreamResult(fos));
        IOUtils.closeQuietly(fos);
    }

    /**
     * 从 user.xml 文件 unmarshalling 生成 User 的对象
     */
    @Test
    public void testUnmarshalling() throws Exception {
        FileInputStream fis = FileUtils.openInputStream(new File(FILE_NAME));
        User user = (User)marshaller.unmarshal(new StreamSource(fis));
        System.out.println(JSON.toJSONString(user));
        IOUtils.closeQuietly(fis);
    }
}
```

运行 `testMarshalling()` 生成文件 `/Users/Biao/Desktop/user.xml`，内容如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<user>
    <user_id>12</user_id>
    <username>Alice</username>
    <password>Passw0rd</password>
</user>
```

运行 `testUnmarshalling()` 输出

```
{"id":12,"password":"Passw0rd","username":"Alice"}
```

## 问题
上面例子中 User 的属性都是简单属性，没有数组，List，Map，Set，复杂对象等，在网络上找到的资料基本都是用的简单属性，实际开发中肯定是不够用的，还需要进一步研究相关内容。
