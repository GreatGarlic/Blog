---
title: JNDI 数据源
date: 2016-11-17 19:37:01
tags: Misc
---

本文基于 Spring 的 `JdbcTemplate` 来介绍使用 JNDI 数据源。

**JNDI 有两种配置方式:**

* 全局 JNDI 配置 - 在 `<tomcat>/conf/server.xml` 里配置，所有项目都能使用
* 局部 JNDI 配置 - 有两种方式，只有配置此 JNDI 的项目自己能使用
    * 在 `META-INF/context.xml` 里配置
    * 在 `<tomcat>/conf/Catalina/localhost/<projectRelated>.xml` 里配置

<!--more-->

## 全局 JNDI 配置
在 `<tomcat>/conf/server.xml` 中的 `<GlobalNamingResources>` 下添加 `<Resource>` 如下配置 JNDI 数据源:

```xml
<GlobalNamingResources>
    <!-- Existing elements -->
    
    <Resource name="jdbc/xtuerDB"
        auth="Container"
        type="javax.sql.DataSource"
        driverClassName="com.mysql.jdbc.Driver"
        url="jdbc:mysql://localhost:3306/test?useUnicode=true&amp;characterEncoding=UTF-8"
        username="root"
        password="root"
        maxActive="50"
        maxIdle="30"
        maxWait="10000" /> 
</GlobalNamingResources>
```

## 局部 JNDI 配置
在 `META-INF/context.xml` 中如下配置 JNDI 数据源:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Context>
    <Resource name="jdbc/xtuerDB"
        auth="Container"
        type="javax.sql.DataSource"
        driverClassName="com.mysql.jdbc.Driver"
        url="jdbc:mysql://localhost:3306/test?useUnicode=true&amp;characterEncoding=UTF-8"
        username="root"
        password="root"
        maxActive="50"
        maxIdle="30"
        maxWait="10000" />
</Context>
```

在 `<tomcat>/conf/Catalina/localhost/<projectRelated>.xml` 里配置:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Context reloadable="false" path="/avatar" override="true"
    docBase="/Projects/Avatar/target/Avatar-1.0">
    <Resource name="jdbc/xtuerDB"
        auth="Container"
        type="javax.sql.DataSource"
        driverClassName="com.mysql.jdbc.Driver"
        url="jdbc:mysql://localhost:3306/test?useUnicode=true&amp;characterEncoding=UTF-8"
        username="root"
        password="root"
        maxActive="50"
        maxIdle="30"
        maxWait="10000" />
</Context>
```

## 使用 JNDI 数据源
1. 按上面的说明选择一种适合项目的方式配置好 JNDI 的数据源
2. 把 MySQL 的 connector 如 mysql-connector-java-5.1.21.jar 放到 `<tomcat>/lib` 下
3. 在 Spring 的配置文件里配置好 DataSource 如文件命名为 database.xml

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">
        <bean id="dataSource" class="org.springframework.jndi.JndiObjectFactoryBean">
            <property name="jndiName">
                <value>java:comp/env/jdbc/xtuerDB</value>
            </property>
        </bean>
        <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
            <property name="dataSource" ref="dataSource"/>
        </bean>
    </beans>
    ```

    > `<value>java:comp/env/jdbc/xtuerDB</value>`  
    > `java:comp/env/` 是固定的前缀  
    > `jdbc/xtuerDB` 是 JNDI 数据源的名字
4. 在 web.xml 中加载 database.xml

    ```xml
    <!-- Context Param -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            /WEB-INF/config/database.xml
        </param-value>
    </context-param>
    
    <!-- Listener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    ```
5. 在 Controller 里使用 jdbcTemplate 访问数据库

    ```java
    @RequestMapping("/users")
    @ResponseBody
    public String getUsers() {
        String sql = "SELECT * from user";
        List<Map<String, Object>> rows = jdbcTemplate.queryForList(sql);
    
        return rows.toString();
    }
    ```

## Gretty 需要启用 JNDI
Gretty 的嵌入式 Tomcat 和 Jetty 默认没有启用 JNDI 和指定 `context.xml`，需要在配置中启用

```groovy
gretty {
    port = 8080
    contextPath = ''
    servletContainer  = 'tomcat7'
    enableNaming      = true // 启用 JNDI
    contextConfigFile = 'tomcat-context.xml'
    serverConfigFile  = 'tomcat.xml'
}
```

> serverConfigFile 配置 Tomcat 的 server.xml  
> contextConfigFile 配置 context.xml

请参考 <http://akhikhl.github.io/gretty-doc/Gretty-configuration.html>

