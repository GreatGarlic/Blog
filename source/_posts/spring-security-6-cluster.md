---
title: Spring Security 集群
date: 2016-04-10 11:23:02
tags: Spring-Security
---

集群的关键点是 session 共享，这里使用 `spring-session-data-redis` 把 session 存储到 Redis 实现集群里 session 的共享，全是配置，不需要修改一行 Java 代码就能实现 Session 的集群共享。

<!--more-->

## Gradle 依赖
```
"org.springframework.session:spring-session-data-redis:1.1.1.RELEASE"
```

## spring-session.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">
    <context:annotation-config/>
    <bean class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory"/>
    <bean class="org.springframework.session.data.redis.config.annotation.web.http.RedisHttpSessionConfiguration"/>
</beans>
```

虽然上面的配置一般情况下足够了，但是很多时候需要配置 JedisPool, Redis Server 的 IP，密码等等，可以参考下面的配置:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">
    <context:annotation-config/>

    <bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig"/>
    <bean id="jedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
        <!--<property name="hostName" value="${redis.host}" />-->
        <!--<property name="port" value="${redis.port}" />-->
        <!--<property name="password" value="${redis.password}" />-->
        <!--<property name="timeout" value="${redis.timeout}" />-->
        <property name="poolConfig" ref="jedisPoolConfig" />
        <property name="usePool" value="true" />
    </bean>

    <bean id="redisTemplate" class="org.springframework.data.redis.core.StringRedisTemplate">
        <property name="connectionFactory" ref="jedisConnectionFactory" />
    </bean>

    <!-- 将session放入redis -->
    <bean id="redisHttpSessionConfiguration"
          class="org.springframework.session.data.redis.config.annotation.web.http.RedisHttpSessionConfiguration">
        <property name="maxInactiveIntervalInSeconds" value="1800" />
    </bean>
</beans>
```

## web.xml
Spring Session Data Redis 使用 Servlet Filter 来实现的，filter-name 必须为 `springSessionRepositoryFilter`，并且必须放在 Spring Security 的 filter `springSecurityFilterChain` 之前(filter 是按照定义的顺序加载的)，web.xml 中加入下面的代码:

```xml
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            classpath:config/spring-session.xml
            classpath:config/spring-security.xml
        </param-value>
    </context-param>

    <!-- Spring Session Data Redis Filter -->
    <filter>
        <filter-name>springSessionRepositoryFilter</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>springSessionRepositoryFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

通过上面的配 就实现了在 Redis 里存储 Session，集群里共享 Session 了，不需要修改任何代码，当然，别忘了启动 Redis 服务器(`redis-servr`)。

## 测试
* Redis 必须在应用前启动
    * 应用启动前没有启动 Redis，应用启动失败，不能处理请求
    * 应用启动前已经启动 Redis，应用启动成功，中途 Redis 关闭了，处理请求会报异常，再次启动 Redis，应用会自动连接 Redis，正常处理请求
* 既然是集群，就需要一个负债均衡如 `Nginx` 来分发请求
* 启动多个应用的实例
* 访问 <http://biao.com/admin>
* 登陆
* 不停的刷新访问 <http://biao.com/admin>
* 看到只需要一次登陆，集群中的所有应用都不需要再次登陆了

## 参考
* [Nginx 负载均衡](http://qtdebug.com/spring-web/Nginx%20负载均衡.html)
