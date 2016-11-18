---
title: MyBatis 集成
date: 2016-10-15 21:35:46
tags: Spring-Web
---
SpringMVC 集成 MyBatis 需要以下几个文件

* 配置文件
    * datasource.xml
    * mybatis.xml
    * web.xml
* 查询需要的文件
    * 可选：bean（也可以不要，直接用 Map）
    * 必要：mapper xml
    * 必要：mapper interface

<!--more-->

## Gradle 依赖
```groovy
compile(
    "org.springframework:spring-jdbc:4.3.0.RELEASE",
    "mysql:mysql-connector-java:5.1.21",         
    "org.mybatis:mybatis:3.2.1",            
    "org.mybatis:mybatis-spring:1.2.2", 
    "com.alibaba:druid:1.0.26" 
)
```

## datasource.xml
保存在 `resources/config/datasource.xml`

> 连接池使用 Druid，可以监控数据库访问的性能，参考文档
> 
> * <https://github.com/alibaba/druid/wiki/常见问题>
> * <https://github.com/alibaba/druid/wiki/DruidDataSource配置> 
> * <https://github.com/alibaba/druid/wiki/DruidDataSource配置属性列表>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- Data Source using Druid. -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
        <property name="url" value="jdbc:mysql://localhost:3306/test?useUnicode=true&amp;characterEncoding=UTF-8"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>

        <property name="maxActive" value="600"/>
        <property name="initialSize" value="1"/>
        <property name="maxWait" value="60000"/>
        <property name="minIdle" value="1"/>

        <property name="timeBetweenEvictionRunsMillis" value="60000"/>
        <property name="minEvictableIdleTimeMillis" value="300000"/>

        <property name="validationQuery" value="select NOW()"/>
        <property name="testWhileIdle" value="true"/>
        <property name="testOnBorrow" value="false"/>
        <property name="testOnReturn" value="false"/>

        <property name="poolPreparedStatements" value="true"/>
        <property name="maxOpenPreparedStatements" value="20"/>
    </bean>
</beans>
```

> **testOnBorrow** 为 false，服务器重启后也会自动重连。  
> **driverClassName** 可以不写，能够根据 url 自动识别 dbType

## mybatis.xml
保存在 `resources/config/mybatis.xml `

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 1. Data Source -->
    <import resource="classpath:config/datasource.xml"/>

    <!-- 2. SQL session factory -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="mapperLocations" value="classpath:mapper/**/*.xml"/> <!-- Mapper xml -->
        <property name="typeAliasesPackage" value="com.xtuer.bean"/>
    </bean>

    <!-- 3. Instantiate Mapper -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.xtuer.mapper"/>
    </bean>
</beans>
```

> `MapperScannerConfigurer` 会为包 `com.xtuer.mapper` 下的每个类在 Spring 容器里创建一个对象，可以使用 `@Autowired` 获取这些对象。
> 
> 如果不使用 `MapperScannerConfigurer`，我们就必须手动的创建 Mapper 的对象。
> 
> 参考: [MyBatis - MyBatis-Spring | 简介](http://mybatis.github.io/spring/zh/)

## web.xml 加载 mybatis 配置
在 web.xml 中用 ContextLoaderListener 加载 mybatis.xml。  
ContextLoaderListener 对应的容器是其他 Spring 容器的父容器，所以在里面创建的 MyBatis 的 mapper 在 springmvc 这个容易以及其他的容器中都能使用。

```xml
<!-- 加载 MyBatis, Spring Security 配置等 -->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>
        classpath:config/mybatis.xml
    </param-value>
</context-param>

<!-- Listener -->
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

## 使用 MyBatis 访问数据库
至此，MyBatis 已经配置好，下面以查询数据库中表 demo 为例，其对应的 bean 为 Demo

> 一般对一个表的查询，需要准备
> 
> * bean
> * mapper interface: 其实例通过 MapperScannerConfigurer 自动创建到 Spring 容器
> * mapper xml: SQL 语句的文件

## 数据表 demo
id  | info
--- | ----
1   | Biao
2   | 海龙

## Demo.java
```java
package com.xtuer.bean;

public class Demo {
    private int id;
    private String info;

    public Demo() {
    }

    public Demo(int id, String info) {
        this.id = id;
        this.info = info;
    }
    
    // Getters and Setters
}
```

## DemoMapper.java
```java
package com.xtuer.mapper;

import com.xtuer.bean.Demo;

public interface DemoMapper {
    public Demo findDemoById(int id);
}
```

## mapper/Demo.xml
保存在 `resources/mapper/Demo.xml`，在这里编写 SQL 语句。

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<!DOCTYPE mapper PUBLIC
        "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--namespace 非常重要：必须是 Mapper 类的全路径-->
<mapper namespace="com.xtuer.mapper.DemoMapper">
    <select id="findDemoById" parameterType="int" resultType="Demo">
        SELECT id, info FROM demo WHERE id = #{id}
    </select>
</mapper>
```

## Controller
```java
@Autowired
private DemoMapper demoMapper;

@GetMapping("/demos/{id}")
@ResponseBody
public Demo findDemoById(@PathVariable int id) {
    return demoMapper.findDemoById(id);
}
```

## 测试
* 访问 <http://localhost:8080/demos/1>

    ```
    控制台输出:
    [2016-10-15 21:54:29] [DEBUG] [BaseJdbcLogger.java-debug:132] - ==>  Preparing: SELECT id, info FROM demo WHERE id = ?
    [2016-10-15 21:54:29] [DEBUG] [BaseJdbcLogger.java-debug:132] - ==> Parameters: 1(Integer)
    
    网页输出:
    {"id":1,"info":"Biao"}
    ```

* 访问 <http://localhost:8080/demos/2> 输出

    ```
    控制台输出:
    [2016-10-15 21:54:33] [DEBUG] [BaseJdbcLogger.java-debug:132] - ==>  Preparing: SELECT id, info FROM demo WHERE id = ?
    [2016-10-15 21:54:33] [DEBUG] [BaseJdbcLogger.java-debug:132] - ==> Parameters: 2(Integer)
    
    网页输出:
    {"id":2,"info":"海龙"}
    ```

## MyBatis 相关日志
* 关闭 MyBatis 日志，修改 logback.xml

    ```xml
    <logger name="org.mybatis" level="debug"/> 修改为 <logger name="org.mybatis" level="off"/>
    ```

* 关闭 MyBatis 输出的 SQL 语句，修改 logback.xml

    ```xml 
    <logger name="com.xtuer.mapper" level="debug"/> 修改为 <logger name="com.xtuer.mapper" level="off"/>
    ```

## Mapper xml 样例文件
```xml
<?xml version="1.0" encoding="UTF-8" ?>

<!DOCTYPE mapper PUBLIC
        "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >

<mapper namespace="com.tur.mapper.UserMapper" >
    <sql id="columns" > id, age, name</sql>

    <!-- [[1]] 简单的 JavaBean，直接使用 resultType: 数据库表的列与 JavaBean 的属性对应 -->
    <select id="selectUserById" parameterType="int" resultType="com.tur.domain.User" >
        SELECT <include refid="columns"/>
        FROM user WHERE id = #{id}
    </select>

    <select id="selectUsersByName" parameterType="string" resultType="com.tur.domain.User" >
        SELECT <include refid="columns"/>
        FROM user WHERE name = #{name}
    </select>

    <!-- [[2]] 可以使用 resultMap 映射自己的类: 例如多表查询时 -->
    <select id="selectUserById" parameterType="int" resultMap="userResultMap" >
        SELECT <include refid="columns"/>
        FROM user WHERE id = #{id}
    </select>
    <resultMap id="userResultMap" type="com.tur.domain.User" >
        <id property="id" column="id"/>
        <result property="age" column="age"/>
        <result property="name" column="name"/>
    </resultMap>

    <!-- [[3]] 使用 resultMap 映射，属性是另一个类的对象: association -->
    <select id="selectFullUserById" parameterType="int" resultMap="userAssociationResultMap" >
        SELECT
            user.id         as id, <!-- 重命名列非常有用 -->
            user.age        as age,
            user.name       as name,
            ui.id           as user_info_id,
            ui.user_id      as user_info_user_id,
            ui.telephone    as user_info_telephone,
            ui.address      as user_info_address
        <!--FROM user, user_info ui-->
        FROM user
            INNER JOIN user_info ui ON user.id=ui.user_id
        WHERE user.id=#{id}
            <!--AND user.id=ui.user_id-->
    </select>
    <resultMap id="userAssociationResultMap" type="com.tur.domain.User" >
        <id property="id" column="id"/>
        <result property="age" column="age"/>
        <result property="name" column="name"/>
        <!--嵌套映射中还可以使用 resultMap: association, collection
        还可以使用嵌套查询，但是会产生 N＋1 问题，在大数量的数据库里会有很大的性能问题-->
        <!--<association property="userInfo" column="user_info_id" javaType="domain.UserInfo">
            <id     property="id"        column="user_info_id"/>
            <result property="userId"    column="user_info_user_id"/>
            <result property="telephone" column="user_info_telephone"/>
            <result property="address"   column="user_info_address"/>
        </association>-->
        <!--association 是一对一关系，collection 是一对多关系-->
        <!--使用 columnPrefix 可以使 result map 重用-->
        <association property="userInfo" column="user_info_id" columnPrefix="user_info_" resultMap="userInfoResultMap"/>
    </resultMap>
    <resultMap id="userInfoResultMap" type="com.tur.domain.UserInfo" >
        <id     property="id"        column="id"/>
        <result property="userId"    column="user_id"/>
        <result property="telephone" column="telephone"/>
        <result property="address"   column="address"/>
    </resultMap>

    <select id="selectUsersWithName" parameterType="list" resultType="com.tur.domain.User" >
        SELECT  <include refid="columns"/>
        FROM    user
        WHERE   name in
        <foreach item="item" index="index" open="("separator=","close=")" collection="list" >
            #{item}
        </foreach>
    </select>
</mapper>
```

> 一对一使用 `association`  
> 一对多使用 `collection`

## 使用 DBCP2 作为数据源
```groovy
compile 'org.apache.commons:commons-dbcp2:2.1.1'
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- Data Source using DBCP. -->
    <bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/test?useUnicode=true&amp;characterEncoding=UTF-8"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>

        <!-- 连接池启动时的初始值 -->
        <property name="initialSize" value="10"/>
        <!-- 连接池的最大值 -->
        <property name="maxTotal" value="100"/>
        <!-- 最大空闲值.当经过一个高峰时间后，连接池可以慢慢将已经用不到的连接慢慢释放一部分，一直减少到maxIdle为止 -->
        <property name="maxIdle" value="50"/>
        <!-- 最小空闲值.当空闲的连接数少于阀值时，连接池就会预申请去一些连接，以免洪峰来时来不及申请 -->
        <property name="minIdle" value="5"/>

        <property name="poolPreparedStatements"    value="true"/>
        <property name="maxOpenPreparedStatements" value="10"/>

        <!-- 给出一条简单的sql语句进行验证 -->
        <property name="validationQuery" value="select NOW()"/>
        <!-- 在取出连接时进行有效验证, 实现如服务器重启后自动重连 -->
        <property name="testOnBorrow"  value="false"/>
        <property name="testWhileIdle" value="true"/>
        <property name="logAbandoned"  value="true"/>
        <property name="removeAbandonedTimeout" value="120"/>
        <!-- 运行判断连接超时任务的时间间隔，单位为毫秒，默认为-1，即不执行任务 -->
        <property name="timeBetweenEvictionRunsMillis" value="3600000"/>
        <!-- 连接的超时时间，默认为半小时 -->
        <property name="minEvictableIdleTimeMillis" value="3600000"/>
    </bean>
</beans>
```

> 如果 testOnBorrow 为 false，服务器重启后链接都会无效，访问数据库就会报错，为了实现服务器重启后能够自动重连，需要把 testOnBorrow 设置为 true。
