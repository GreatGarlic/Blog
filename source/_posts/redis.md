---
title: Java 访问 Redis
date: 2016-04-06 14:38:34
tags: [Spring, Redis]
---

Redis 是一个开源的使用 ANSI C 语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value 数据库，并提供多种语言的 API，和 Memcached 类似，它支持存储的 value 类型相对更多，包括string(字符串)、list(链表)、set(集合)、zset(sorted set -- 有序集合)和 hash(哈希类型)。

<!--more-->

使用 `spring-data-redis` 访问 Redis 很方便，还能支持自动重连的机制。

## Gradle 依赖
```
dependencies {
    compile 'org.springframework.data:spring-data-redis:1.6.4.RELEASE'
    compile 'redis.clients:jedis:2.8.1'
}
```

## Spring Bean 配置文件 `redis.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
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
</beans>
```

## 访问 Redis
```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.data.redis.core.RedisTemplate;

import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class RedisTemplateTest {
    static int count = 0;

    public static void main(String[] args) throws Exception {
        ApplicationContext context = new ClassPathXmlApplicationContext("redis.xml");
        final RedisTemplate redisTemplate = context.getBean("redisTemplate", RedisTemplate.class);

        redisTemplate.opsForValue().set("name", "Biao"); // 设置数据到 Redis
        redisTemplate.opsForValue().set("nginx", "Go");
        System.out.println(redisTemplate.keys("n*")); // 使用模式匹配查找以 n 开头的所有 key

        // 每隔一秒钟从 Redis 里读取一次数据
        Executors.newScheduledThreadPool(1).scheduleAtFixedRate(new Runnable() {
            @Override
            public void run() {
                try {
                    System.out.println((++count) + ":" + redisTemplate.opsForValue().get("name"));
                } catch(Exception ex) { }
            }
        }, 0, 1, TimeUnit.SECONDS);
    }
}
```

## 测试
1. 启动 Redis 服务器: `redis-server`
2. 运行上面的程序: `java RedisTemplateTest`，输出

    > [name, nginx]  
    > 1:Biao  
    > 2:Biao  
3. 停止 Redis 服务器，Redis 链接断开，控制台没有输出
4. 启动 Redis 服务器，Redis 自动重连，控制台继续输出
