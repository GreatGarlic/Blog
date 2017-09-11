---
title: SpringBoot MyBatis
date: 2017-09-11 14:31:15
tags: SpringBoot
---

SpringBoot 使用 MyBatis 主要为以下 4 步:

1. 引入依赖

   ```groovy
   compile('org.mybatis.spring.boot:mybatis-spring-boot-starter:1.3.1')
   runtime('mysql:mysql-connector-java')
   ```

2. 配置数据源: 配置 **application.properties**

   ```ini
   spring.datasource.username=root
   spring.datasource.password=root
   spring.datasource.url=jdbc:mysql://localhost:3306/test
   spring.datasource.driver-class-name=com.mysql.jdbc.Driver
   ```

3. 编写 Mapper: 使用注解 **@Mapper** 自动生成 Mapper 对象

   ```java
   package com.xtuer.mapper;

   import org.apache.ibatis.annotations.Mapper;
   import org.apache.ibatis.annotations.Select;

   import java.util.Map;

   @Mapper
   public interface UserMapper {
       @Select("SELECT * FROM user WHERE username=#{username}")
       public Map findUserByUsername(String username);
   }
   ```

4. 使用 Mapper: 使用 **@Autowired** 装配 mapper

   ```java
   package com.xtuer.controller;

   import com.xtuer.mapper.UserMapper;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RestController;

   import java.util.Map;

   @RestController
   public class HelloController {
       @Autowired
       private UserMapper userMapper;

       @GetMapping("/hello")
       public Map hello(@RequestParam String username) {
           return userMapper.findUserByUsername(username);
       }
   }
   ```

   <!--more-->

## Mapper XML

上面的 SQL 语句写在了 Mapper 中，更多时候是希望写到 xml 文件方便管理以及修改，在 application.properties 中加上

```ini
# Mapper XML 文件的路径
mybatis.mapper-locations=classpath:mapper/*.xml
# 不需要写类的全路径了，例如 com.xtuer.bean.User 在 Mapper XML 中可省区包名，简写为 User
mybatis.type-aliases-package=com.xtuer.bean
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<!DOCTYPE mapper PUBLIC
        "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--namespace 非常重要：必须是 Mapper 类的全路径-->
<mapper namespace="com.xtuer.mapper.UserMapper">
    <select id="findUserByUsername" parameterType="string" resultType="User">
        SELECT user_id as userId, username, password FROM user WHERE username = #{username}
    </select>
</mapper>
```

> 注: UserMapper 中的 @Select 去掉，再写一个 User bean，就不在此赘述了.

## 连接池

使用连接池 Druid 需要引入依赖 `compile('com.alibaba:druid-spring-boot-starter:1.1.3')`，以及 application.properties 中配置

```ini
spring.datasource.druid.initial-size=2
spring.datasource.druid.max-active=150
spring.datasource.druid.min-idle=20
spring.datasource.druid.max-wait=60000
spring.datasource.druid.pool-prepared-statements=true
spring.datasource.druid.max-pool-prepared-statement-per-connection-size=20
spring.datasource.druid.max-open-prepared-statements= 20
spring.datasource.druid.validation-query=SELECT 1
spring.datasource.druid.validation-query-timeout=2000
spring.datasource.druid.test-on-borrow=false
spring.datasource.druid.test-on-return=false
spring.datasource.druid.test-while-idle=true
spring.datasource.druid.time-between-eviction-runs-millis=60000
spring.datasource.druid.min-evictable-idle-time-millis=300000
spring.datasource.druid.max-evictable-idle-time-millis=300000
```

更多帮助请参考 <https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter>

> MySQL 查看活跃连接数: `SHOW PROCESSLIST`

