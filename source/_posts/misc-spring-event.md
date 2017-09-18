---
title: Spring Event
date: 2016-11-21 18:30:05
tags: [Misc, SpringCore]
---
JMS 可用于在不同应用之间通讯，而应用系统内部对象之间的通讯可以使用 Spring Event 来实现（JMS 也能实现同样的效果）。

**Spring Event 的关键对象为 `事件`、`事件发送者` 和 `事件监听器`:**

* 事件类需要继承 ApplicationEvent
* 事件发送者需要实现接口 ApplicationEventPublisherAware，Spring 容器在创建事件发送者对象时因为发现它实现了接口 ApplicationEventPublisherAware，就会自动地注入 applicationEventPublisher，这个对象是真正的用于发送事件的对象
* 事件监听器需要实现接口 ApplicationListener，在 Spring 容器里注册事件监听器(其实就是生成一个对象，Spring 会自动识别它是否为事件监听器)

<!--more-->

项目目录:

```
├── main
│   ├── java
│   │   ├── Main.java
│   │   └── com
│   │       └── xtuer
│   │           ├── bean
│   │           │   └── User.java
│   │           └── event
│   │               ├── event
│   │               │   └── UserEvent.java
│   │               ├── listener
│   │               │   └── UserEventListener.java
│   │               └── publisher
│   │                   └── UserEventPublisher.java
│   └── resources
│       └── spring-beans.xml
└── test
    ├── java
    └── resources
```

## Gradle 依赖
有 ApplicationContext 实现的包就可以使用 Spring 事件，这里使用了 spring-webmvc

```groovy
compile 'org.springframework:spring-webmvc:4.3.0.RELEASE'
compile 'org.projectlombok:lombok:1.16.10'
compile 'com.alibaba:fastjson:1.2.17'
```

## 事件 UserEvent
```java
package com.xtuer.event.event;

import com.xtuer.bean.User;
import lombok.Getter;
import lombok.Setter;
import org.springframework.context.ApplicationEvent;

@Getter
@Setter
public class UserEvent extends ApplicationEvent {
    private String eventType;
    private User user;

    /**
     * Create a new ApplicationEvent.
     *
     * @param source the object on which the event initially occurred (never {@code null})
     */
    public UserEvent(Object source, String eventType, User user) {
        super(source);
        this.eventType = eventType;
        this.user = user;
    }
}
```

## 事件发送者 UserEventPublisher
```java
package com.xtuer.event.publisher;

import com.xtuer.bean.User;
import com.xtuer.event.event.UserEvent;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.context.ApplicationEventPublisherAware;

public class UserEventPublisher implements ApplicationEventPublisherAware {
    private ApplicationEventPublisher applicationEventPublisher;

    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        // Spring 容器在创建 UserEventPublisher 对象时自动注入 applicationEventPublisher
        this.applicationEventPublisher = applicationEventPublisher;
    }

    public void publish(User user, String action) {
        applicationEventPublisher.publishEvent(new UserEvent(this, action, user));
    }
}
```

## 事件监听器 UserEventListener
```java
package com.xtuer.event.listener;

import com.alibaba.fastjson.JSON;
import com.xtuer.event.event.UserEvent;
import org.springframework.context.ApplicationListener;

public class UserEventListener implements ApplicationListener<UserEvent> {
    public void onApplicationEvent(UserEvent event) {
        System.out.printf("Event received: %s - %s\n", event.getEventType(), JSON.toJSONString(event.getUser()));
    }
}
```

## User
```java
package com.xtuer.bean;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class User {
    private int id;
    private String username;
    private String password;

    public User() {
    }

    public User(int id, String username, String password) {
        this.id = id;
        this.username = username;
        this.password = password;
    }
}
```

## Spring 配置文件
spring-beans.xml 注册事件发送者和监听器

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 事件发送者 -->
    <bean id="userEventPublisher" class="com.xtuer.event.publisher.UserEventPublisher"/>

    <!-- 事件监听器 -->
    <bean class="com.xtuer.event.listener.UserEventListener"/>
</beans>
```

## 测试
```java
import com.xtuer.bean.User;
import com.xtuer.event.publisher.UserEventPublisher;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Main {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("classpath:spring-beans.xml");
        UserEventPublisher publisher = context.getBean("userEventPublisher", UserEventPublisher.class);

        User user = new User(1, "Alice", "12345678");
        publisher.publish(user, "CREATE_USER");

        user.setPassword("Passw0rd");
        publisher.publish(user, "UPDATE_USER");
    }
}
```

**上面代码的逻辑为:**

1. 启动 Spring 容器
2. 获取事件发送对象
3. 发送消息
4. 可以看到控制台里输出了下面的内容，说明事件 UserEvent 被事件监听器收到了:

    ```
    Event received: CREATE_USER - {"id":1,"password":"12345678","username":"Alice"}
    Event received: UPDATE_USER - {"id":1,"password":"Passw0rd","username":"Alice"}
    ```

