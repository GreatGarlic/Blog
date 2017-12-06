---
title: Gradle 文件动态内容替换
date: 2016-06-29 10:44:35
tags: [Misc, Gradle]
---

不同环境下的配置文件不一样，为每个环境单独写一套配置文件不易维护，使用 Gradle 能动态的替换文件中预先定义好的占位符生成特定环境下的配置文件并打包输出。

<!--more-->

## 1. src/main/resources/config/`datasource.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- Data Source using DBCP. -->
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="@database.driverClassName@" />
        <property name="url"      value="@database.url@" />
        <property name="username" value="@database.username@" />
        <property name="password" value="@database.password@" />
        ...
    </bean>
</beans>

```

`@database.username@`, `@database.password@` 等是占位符，Gradle 会将其替换掉。

## 2. 定义不同环境下的配置 `config.groovy`

`config.groovy` 和 `build.gradle` 在同一个目录下，定义了不同环境下的配置信息

```java
environments {
    development { // 开发环境
        database {
            driverClassName = 'com.mysql.jdbc.Driver'
            url = 'jdbc:mysql://localhost:3306/survey?useUnicode=true&amp;characterEncoding=UTF-8'
            username = 'root'
            password = 'root'
        }
    }

    production { // 线上环境
        database {
            driverClassName = 'com.mysql.jdbc.Driver'
            url = 'jdbc:mysql://localhost:3306/survey?useUnicode=true&amp;characterEncoding=UTF-8'
            username = 'root'
            password = 'wxyz'
        }
    }
}
```

> `environments` 的内容使用 `ConfigSlurper` 读取

## 3. build.gradle
在 build.gradle 中定义替换的任务 `processResources`

```java
import org.apache.tools.ant.filters.ReplaceTokens

ext {
    // 运行和打包的环境选择, 默认是开发环境
    // 获取 gradle 参数中 -Dprofile 的值: gradle -Denv=production clean build
    environment = System.getProperty("env", "development")
}

def loadConfiguration() {
    println "==> Load configuration for '" + environment + "'"
    def configFile = file('config.groovy') // 配置文件
    return new ConfigSlurper(environment).parse(configFile.toURI().toURL()).toProperties()
}

processResources {
    // src/main/resources 下的文件中 @key@ 的内容使用 config.groovy 里对应的进行替换
    from(sourceSets.main.resources.srcDirs) {
        filter(ReplaceTokens, tokens: loadConfiguration())
    }
}
```

> Use the `ConfigSlurper` to read in properties for our project. The ConfigSlurper supports environments where we can define values for properties per environment.

## 4. 打包
1. `gradle clean build` 或者 `gradle clean -Denv=development build` 生成 `开发环境` 下的配置并打包

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="
                http://www.springframework.org/schema/beans
                http://www.springframework.org/schema/beans/spring-beans.xsd">

        <!-- Data Source using DBCP. -->
        <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
            <property name="driverClassName" value="com.mysql.jdbc.Driver" />
            <property name="url" value="jdbc:mysql://localhost:3306/survey?useUnicode=true&amp;characterEncoding=UTF-8" />
            <property name="username" value="root" />
            <property name="password" value="root" />

            <!-- 连接池启动时的初始值 -->
            <property name="initialSize" value="10" />
            ...
        </bean>
    </beans>
    ```
2. `gradle clean -Denv=production build` 生成 `线上环境` 的配置并打包

    ```xml 
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="
                http://www.springframework.org/schema/beans
                http://www.springframework.org/schema/beans/spring-beans.xsd">

        <!-- Data Source using DBCP. -->
        <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
            <property name="driverClassName" value="com.mysql.jdbc.Driver" />
            <property name="url" value="jdbc:mysql://localhost:3306/survey?useUnicode=true&amp;characterEncoding=UTF-8" />
            <property name="username" value="root" />
            <property name="password" value="wxyz" />

            <!-- 连接池启动时的初始值 -->
            <property name="initialSize" value="10" />
            ...
        </bean>
    </beans>   
    ```

## 参考
[Gradle在大型Java项目上的应用](http://www.infoq.com/cn/articles/Gradle-application-in-large-Java-projects/)
