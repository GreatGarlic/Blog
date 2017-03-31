---
title: Miscellaneous
date: 2017-03-31 19:12:57
tags: Spring-Web
---
## RequestMapping

> Since introduction of `RequestMappingHandlerMapping` and `RequestMappingHandlerAdapter` in Spring 3.1 the distinction is even simpler: RequestMappingHandlerMapping finds the appropriate handler method for the given request. RequestMappingHandlerAdapter executes this method, providing it with all the arguments.

## mvc:view-controller

有很多静态页面，里没有动态的内容，如果写 Controller 去做映射的话又感觉很麻烦，都是体力活，没什么意思，这时可以用 `mvc:view-controller` 进行映射达到相同的效果而又不需要写 Controller。

```js
<!-- result.htm 是 View 的名字 -->
<mvc:view-controller path="/xtuer" view-name="result.htm"/>
```

访问 **http://localhost/xtuer**，则 View Resolver 访问的是 **/WEB-INF/view/ftl/result.htm**。

## Import Resource

把 Spring 的 Bean Configuration File 根据模块分散到不同的文件里，便于管理。
例如把上面提到的 `mvc:view-controller`，把所有静态文件的映射都放到 spring-static.xml 里，在 SpringMVC 主配置文件 spring-mvc.xml `import` spring-static.xml

```js
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc.xsd">
    ...
    <import resource="classpath:spring-static.xml"/> <!--位置随意-->
    ...
</beans>
```

```js
<!-- 文件名: spring-static.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc.xsd">
    ...   
    <mvc:view-controller path="/xtuer" view-name="result.htm"/>
    ...
</beans>
```

## PropertyPlaceholderConfigurer

目的：在 properties 文件里定义一些属性，在 spring 的配置文件里用 `${propertyName}` 的方式访问属性值，使用 `PropertyPlaceholderConfigurer` 加载 properties 文件。

**数据源的配置:**

```js
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd">
    ...
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
        <property name="url" value="jdbc:mysql://localhost:3306/test?useUnicode=true&amp;characterEncoding=UTF-8" />
        <property name="username" value="root" />
        <property name="password" value="root" />
    </bean>
    ...
</beans>
```

上面的 Spring 配置对于维护人员来说看起来很可能有困难，有太多他们不需要关心的东西干扰，他们实际关心的只是 driverClassName, url, username, password 等的内容，至于 Spring 配置相关的细节不需要知道。把他们提取到 properties 文件里，就是一些简单的 key/value，维护起来很方便，于是 Spring 的配置如下：

```js
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

Spring 会自动的把 `${jdbc.driverClassName}` 替换为 `com.mysql.jdbc.Driver`，等等。

database.properties 内容如下：

```ini
jdbc.driverClassName=com.mysql.jdbc.Driver
# 在 xml 里 & 要用 &amp; 表示，在 properties 文件里就直接用 &
jdbc.url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=UTF-8
jdbc.username=root
jdbc.password=root
```

修改数据源配置时只需要修改 database.properties 文件，这样就简单很多了，不需要了解 Spring 的配置信息。

**使用 PropertyPlaceholderConfigurer 加载 properties 文件**

```js
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="location">
            <value>classpath:database.properties</value>
        </property>
    </bean>
</beans>
```

**加载多个 properties 文件**

```js
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations">
            <list>
                <value>classpath:database1.properties</value>
                <value>classpath:database2.properties</value>
            </list>
        </property>
    </bean>
</beans>
```

**使用 context:property-placeholder 简化 PropertyPlaceholderConfigurer**

```js
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd">
    <context:property-placeholder location="classpath:database1.properties" ignore-unresolvable="true"/>
    <context:property-placeholder location="classpath:database2.properties" ignore-unresolvable="true"/>
    
    <!--推荐使用下面这种方式: 一个项目(可有多模块)或一个系统的配置应该放在一起，不宜分散-->
    <context:property-placeholder location="classpath*:conf/conf*.properties"/>
</beans>
```

