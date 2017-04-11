---
title: Spring Http 远程方法调用
date: 2017-04-11 09:18:25
tags: Misc
---
Spring Http 远程调用 (Spring Http Remote Invocation) 是 Spring 提供的一种特殊的允许通过 HTTP 进行 Java 串行化的远程调用策略，**支持任意 Java 接口**，相对应的支持类是 HttpInvokerServiceExporter (服务器端) 和 HttpInvokerProxyFactoryBean (客服端)。

**远程调用两部分:**

* 服务端
* 客户端

**调用逻辑:**

![](/img/misc/SpringHttpInvoker.png)

<!--more-->

## 服务端实现

1. 先创建一个 SpringMVC 的 Maven Web 项目
2. 定义一个提供服务的接口 TimeService

    ```java
    package com.xtuer.service;

    public interface TimeService {
        String getTime();
    }
    ```
3. 实现接口 TimeService 的类为 TimeServiceImpl

    ```java
    package com.xtuer.service;

    public class TimeServiceImpl implements TimeService {
        @Override
        public String getTime() {
            return "当前时间是: " + System.nanoTime();
        }
    }
    ```
4. Spring Http 远程调用的配置文件 spring-http-remote.xml， 定义服务的 URI 和提供服务的类对象

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE beans PUBLIC "-//SPRING/DTD BEAN/EN"
            "http://www.springframework.org/dtd/spring-beans.dtd">
    <beans>
        <bean name="timeService" class="com.xtuer.service.TimeServiceImpl" />
        <bean name="/remote/time-service" class="org.springframework.remoting.httpinvoker.HttpInvokerServiceExporter">
            <property name="serviceInterface" value="com.xtuer.service.TimeService" />
            <property name="service" ref="timeService" />
        </bean>
    </beans>
    ```
5. 在 web.xml 里配置一个 Servlet 用于提供 URL 给客户端访问上面定义的服务

    ```xml
    <!-- Spring Http Remote Invocation -->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:config/spring-http-remote.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>    
    ```
    > 远程调用访问这个服务的 URL 为：http://host:port/remote/time-service，浏览器里直接访问这个 URL 会报异常 **java.io.EOFException: null**。
    > **/remote/time-service** 为 spring-http-remote.xml 里面定义的 Bean HttpInvokerServiceExporter 的 name。

## 客户端实现

1. 提供一个 TimeService 的接口，和服务器端的接口一模一样

    ```java
    package com.xtuer.service;

    public interface TimeService {
        String getTime();
    }
    ```
2. 配置文件 spring-http-remote.xml 定义了提供服务的 URL 和使用的接口 TimeService

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE beans PUBLIC "-//SPRING/DTD BEAN/EN"
            "http://www.springframework.org/dtd/spring-beans.dtd">
    <beans>
        <bean id="timeService" class="org.springframework.remoting.httpinvoker.HttpInvokerProxyFactoryBean">
            <property name="serviceUrl" value="http://localhost:8080/remote/time-service" />
            <property name="serviceInterface" value="com.xtuer.service.TimeService" />
        </bean>
    </beans>
    ```
3. 访问服务

    ```java
    package com.xtuer.remote;

    import com.xtuer.service.TimeService;
    import org.junit.Test;
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;

    public class TestSpringHttpRemote {
        @Test
        public void testRemote() {
            ApplicationContext context = new ClassPathXmlApplicationContext("config/spring-http-remote.xml");
            TimeService service = context.getBean("timeService", TimeService.class);

            // 远程调用访问服务器端提供的服务
            System.out.println(service.getTime());
            System.out.println(service.getTime());
        }
    }   
    ```

## 运行测试
1. 启动 Web 项目: `mvn tomcat7:run`
2. 运行测试类 TestSpringHttpRemote
3. 输出:

    > 当前时间是: 131708937075732  
    > 当前时间是: 131708984368845

## 参考
* [Spring 中 HttpInvoker 远程方法调用总结](http://www.open-open.com/lib/view/open1408957290478.html)
* [Using Spring HTTP Invoker](http://anindya-bandopadhyay.blogspot.com/2014/08/using-spring-http-invoker.html)