---
title: Java 读取 Properties 和 Yaml Properties
date: 2016-07-03 20:04:12
tags: [Java, Util]
---

Java 可以使用 `PropertiesConfiguration` 来读取 properties 属性文件，Spring 4.3 后还支持了 `Yaml 格式的属性文件`

* PropertiesConfiguration: 读取时可以自动进行类型转换，可以给定默认值
* Yaml 格式的属性文件: 可以使用树形结构，方便分组，比 `.properties` 属性文件更灵活，但是以普通的 `java.util.Properties` 来读取

<!--more-->

## Gradle 依赖
```groovy
compile 'org.springframework:spring-context:4.3.0.RELEASE'
compile 'org.yaml:snakeyaml:1.17'
compile 'commons-configuration:commons-configuration:1.10'

testCompile 'org.springframework:spring-test:4.3.0.RELEASE'
testCompile 'junit:junit:4.12'
```

## 属性文件
`config.properties`

```
username=Dr. Alice
age=22
```

`config.yml`

```
#mysql
mysql:
    jdbc:
        url: jdbc:mysql://localhost:3306
        dirverClass: com.mysql.jdbc.Driver
        username: root
        password: root
username: Dr. Alice
```

## Spring Bean 配置文件 
`spring-beans-config.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="yamlProperties" class="org.springframework.beans.factory.config.YamlPropertiesFactoryBean">
        <property name="resources">
            <list>
                <value>classpath:config.yml</value>
            </list>
        </property>
    </bean>

    <bean id="propertiesConfig" class="org.apache.commons.configuration.PropertiesConfiguration">
        <constructor-arg value="config.properties"/> <!-- 不要用 classpath: -->
    </bean>
</beans>
```

## 测试案例
```java
import org.apache.commons.configuration.PropertiesConfiguration;
import org.junit.runner.RunWith;
import org.junit.Test;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringRunner;

import javax.annotation.Resource;
import java.util.Properties;

@RunWith(SpringRunner.class)
@ContextConfiguration({"classpath:spring-beans-config.xml"})
public class TestYamlPropertiesAndPropertiesConfig {
    @Resource(name = "yamlProperties")
    private Properties yamlProperties;

    @Resource(name = "propertiesConfig")
    private PropertiesConfiguration propertiesConfig;

    @Test
    public void testYamlProperties() {
        System.out.println(yamlProperties.getProperty("mysql.jdbc.url"));
        System.out.println(yamlProperties.getProperty("username"));
    }

    @Test
    public void testPropertiesConfig() {
        System.out.println(propertiesConfig.getString("username"));
        System.out.println(propertiesConfig.getInteger("age", 0));
    }
}
```

输出:

```
Dr. Alice
22
jdbc:mysql://localhost:3306
Dr. Alice
```
