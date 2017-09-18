---
title: Redis 集成
date: 2016-10-16 07:27:45
tags: SpringWeb
---
如果数据每次都从数据库查询，当并发大的时候，性能就会急剧的下降，可以采用 Redis 作为缓存，数据首先从 Redis 里查询，如果 Redis 里没有，然后才从数据库查询，并把查询到的结果放入 Redis。

<!--more-->

## Gradle 依赖
```groovy
compile 'org.springframework.data:spring-data-redis:1.7.4.RELEASE'
compile 'org.springframework.session:spring-session-data-redis:1.2.2.RELEASE'
```

> 为了方便且以后可能需要在集群里使用 Redis 存放 Session，故引入了 `spring-session-data-redis`  
> 同时也指定了使用更新的 `spring-data-redis`

## redis.xml
存放在 `resources/config/redis.xml`

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
        <property name="database" value="0"/>
        <property name="poolConfig" ref="jedisPoolConfig"/>
        <property name="usePool" value="true"/>
    </bean>

    <bean id="redisTemplate" class="org.springframework.data.redis.core.StringRedisTemplate">
        <property name="connectionFactory" ref="jedisConnectionFactory"/>
    </bean>
</beans>
```
> `redisTemplate` 用来访问 Redis 

## 引入 redis.xml
在 spring-mvc.xml 中引入 redis.xml

```xml
<import resource="classpath:config/redis.xml"/>
```

## 使用 RedisTemplate 访问 Redis
```java
@Autowired
private DemoMapper demoMapper;

@Autowired
private StringRedisTemplate redisTemplate;

@GetMapping("/demos/{id}")
@ResponseBody
public Demo findDemoById(@PathVariable int id) {
    // [1] 先从 Redis 查找
    // [2] 找到则返回
    // [3] 找不到则从数据库查询，查询结果放入 Redis
    Demo d = null;
    String redisKey = "demo_" + id;
    String json = redisTemplate.opsForValue().get(redisKey);

    if (json != null) {
        // 如果解析发生异常，有可能是 Redis 里的数据无效，故把其从 Redis 删除
        try {
            d = JSON.parseObject(json, Demo.class);
        } catch (Exception ex) {
            logger.warn(ex.getMessage());
            redisTemplate.delete(redisKey);
        }
    }

    if (d == null) {
        d = demoMapper.findDemoById(id);

        if (d != null) { // 这里需要考虑，null 对象如果不放缓存，如果这个对象被大量访问，会导致缓存穿透，增加数据库的压力
            redisTemplate.opsForValue().set(redisKey, JSON.toJSONString(d));
        }
    }

    return d;
}
```

## 清空 Redis 里的数据
```
flushdb
```

## 查看 Redis 缓存是否生效
* 打印日志
* 查看数据库链接的非活跃时间，查询到了数据，但是非活跃时间越来越长，说明缓存生效了
    * `MySQL` 执行 `show full processlist`  
      ![](/img/spring-web/mysql-connection-status.png)

## 重构
观察从 Redis 取数据，没有再从数据库取，然后放入 Redis 的代码，除了 `d = JSON.parseObject(json, Demo.class) 和 d = demoMapper.findDemoById(id)` 这二句会根据不同的业务逻辑不同外，其他的代码都是不变的，所以可以使用策略模式进行重构。

```java
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.TypeReference;
import org.springframework.data.redis.core.StringRedisTemplate;

import java.util.function.Supplier;

public class RedisDao {
    private StringRedisTemplate redisTemplate;

    /**
     * 缓存优先读取 JavaBean
     *
     * @param redisKey key
     * @param clazz 实体类型
     * @param supplier 缓存失败时的数据提供器， supplier == null 时 return null
     * @param <T>   类型约束
     * @return 实体对象
     */
    public <T> T get(String redisKey, Class<T> clazz, Supplier<T> supplier) {
        T d = null;
        String json = redisTemplate.opsForValue().get(redisKey);

        if (json != null) {
            // 如果解析发生异常，有可能是 Redis 里的数据无效，故把其从 Redis 删除
            try {
                d = JSON.parseObject(json, clazz);
            } catch (Exception ex) {
                redisTemplate.delete(redisKey);
            }
        }

        if (d == null && supplier != null) {
            d = supplier.get();

            // 这里需要考虑，null 对象如果不放缓存，如果这个对象被大量访问，会导致缓存穿透，增加数据库的压力
            if (d != null) {
                redisTemplate.opsForValue().set(redisKey, JSON.toJSONString(d));
            }
        }

        return d;
    }

    /**
     * 缓存优先读取 Collections Or Map
     *
     * @param redisKey key
     * @param typeReference 反序列化集合时 FastJson 需要用 TypeReference 来指定类型，例如类型为 List<Demo>
     * @param supplier 缓存失败时的数据提供器，supplier == null 时 return null
     * @param <T>   类型约束
     * @return 实体对象
     */
    public <T> T get(String redisKey, TypeReference<T> typeReference, Supplier<T> supplier) {
        T d = null;
        String json = redisTemplate.opsForValue().get(redisKey);

        if (json != null) {
            // 如果解析发生异常，有可能是 Redis 里的数据无效，故把其从 Redis 删除
            try {
                d = JSON.parseObject(json, typeReference);
            } catch (Exception ex) {
                redisTemplate.delete(redisKey);
            }
        }

        if (d == null && supplier != null) {
            d = supplier.get();

            // 这里需要考虑，null 对象如果不放缓存，如果这个对象被大量访问，会导致缓存穿透，增加数据库的压力
            if (d != null) {
                redisTemplate.opsForValue().set(redisKey, JSON.toJSONString(d));
            }
        }

        return d;
    }


    public StringRedisTemplate getRedisTemplate() {
        return redisTemplate;
    }

    public void setRedisTemplate(StringRedisTemplate redisTemplate) {
        this.redisTemplate = redisTemplate;
    }
}
```

```java
@GetMapping("/demos/{id}")
@ResponseBody
public Demo findDemoById(@PathVariable int id) {
    String redisKey = "demo_" + id;
    Demo d = redisDao.get(redisKey, Demo.class, () -> demoMapper.findDemoById(id));

    return d;
}
```

不同业务场景可以使用 Lambda 表达式实现 RedisDao.get() 的最后一个参数 Supplier，不需要再关心从 Redis 取数据和存储数据到 Redis 的代码了。
