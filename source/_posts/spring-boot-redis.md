---
title: SpringBoot Redis
date: 2017-09-11 15:59:14
tags: SpringBoot
---

SpringBoot 中使用 Redis 非常简单:

1. 引入依赖

   ```groovy
   compile('org.springframework.boot:spring-boot-starter-data-redis')
   ```

2. 在 application.properties 中配置 Redis

   ```ini
   spring.redis.host=localhost
   spring.redis.port=6379
   spring.redis.password=
   spring.redis.database=0
   spring.redis.pool.max-active=8
   spring.redis.pool.max-wait=-1
   spring.redis.pool.max-idle=8
   spring.redis.pool.min-idle=0
   spring.redis.timeout=0  
   ```

3. 使用 StringRedisTemplate 访问 Redis

   ```java
   @RestController
   public class HelloController {
       @Autowired
       private StringRedisTemplate redisTemplate;

       @GetMapping("/redis")
       public String redis() {
           return redisTemplate.opsForValue().get("user");
       }
   }
   ```

<!--more-->

## 使用 Redis 实现集群

1. 引入依赖

   ```groovy
   compile('org.springframework.session:spring-session-data-redis')
   ```

2. 提供配置类

   ```java
   package com.xtuer;

   import org.springframework.context.annotation.Configuration;
   import org.springframework.session.data.redis.config.annotation.web.http.EnableRedisHttpSession;

   @Configuration
   @EnableRedisHttpSession(maxInactiveIntervalInSeconds = 86400*30)
   public class SessionConfig {
   }
   ```

3. 测试

   ```java
   @GetMapping("/cluster")
   public String cluster(HttpServletRequest request) {
       HttpSession session = request.getSession();
       session.setAttribute("username", "Bob");
       return "23";
   }
   ```

   访问后去 Redis 中看看是否有没有 session 的数据生成(应该是有的).

## 参考资料

请参考 <https://zhuanlan.zhihu.com/p/24977566?refer=dreawer>