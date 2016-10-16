---
title: 处理 Ajax 请求
date: 2016-10-15 12:04:01
tags: Spring-Web
---
SpringMVC 处理 AJAX 请求很简单，只需要在方法的前面加上 `@ResponseBody` 即可。  
Controller 的方法一般返回 String(可以是JSON, XML, 普通的 Text), 也可以是对象。

<!--more-->

## 返回 Json 字符串
1. Controller 中添加方法

    ```java
    @GetMapping("/ajax")
    @ResponseBody // 处理 AJAX 请求，返回响应的内容，而不是 View Name
    public String ajaxString() {
        return "{\"username\": \"Josh\", \"password\": \"Passw0rd\"}";
    }
    ```

2. 访问 <http://localhost:8080/ajax>

    > 输出: `{username: "Josh", password: "Passw0rd"}`

## 自动把对象转换为 Json
使用 `Fastjson` 把对象自动映射为 Json

1. Gradle 依赖

    ```groovy
    compile 'com.alibaba:fastjson:1.2.17'
    ```

2. `spring-mvc.xml` 中把 <mvc:annotation-driven/> 换为下面的配置

    ```xml
    <mvc:annotation-driven>
        <!--enableMatrixVariables="true">-->
        <mvc:message-converters register-defaults="true">
            <!-- StringHttpMessageConverter 编码为UTF-8，防止乱码 -->
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                <constructor-arg value="UTF-8"/>
                <property name="supportedMediaTypes">
                    <list>
                        <bean class="org.springframework.http.MediaType">
                            <constructor-arg index="0" value="text"/>
                            <constructor-arg index="1" value="plain"/>
                            <constructor-arg index="2" value="UTF-8"/>
                        </bean>
                        <bean class="org.springframework.http.MediaType">
                            <constructor-arg index="0" value="*"/>
                            <constructor-arg index="1" value="*"/>
                            <constructor-arg index="2" value="UTF-8"/>
                        </bean>
                    </list>
                </property>
            </bean>

            <bean class="com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter4">
                <property name="supportedMediaTypes">
                    <list>
                        <value>text/html;charset=UTF-8</value>
                        <value>application/json;charset=UTF-8</value>
                    </list>
                </property>
                <property name="fastJsonConfig">
                    <bean class="com.alibaba.fastjson.support.config.FastJsonConfig">
                        <property name="features">
                            <list>
                                <value>AllowArbitraryCommas</value>
                                <value>AllowUnQuotedFieldNames</value>
                                <value>DisableCircularReferenceDetect</value>
                            </list>
                        </property>
                        <property name="dateFormat" value="yyyy-MM-dd HH:mm:ss"></property>
                    </bean>
                </property>
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>
    ```

3. 创建类 Result.java

    ```java
    package com.xtuer.bean;

    public class Result {
        private boolean success;
        private String message;
        private Object data;
    
        public Result(boolean success, String message) {
            this(success, message, null);
        }
    
        public Result(boolean success, String message, Object data) {
            this.success = success;
            this.message = message;
            this.data = data;
        }
        
        // Getters and Setters
    }
    ```

4. Controller 里添加方法

    ```java
    @GetMapping("/ajax-object")
    @ResponseBody
    public Result ajaxObject() {
        return new Result(true, "你好");
    }
    ```

5. 访问 <http://localhost:8080/ajax-object>

    > 输出: `{"message":"你好","success":true}`

## 参考资料
* [SpringMVC fastjson 与 Jackson 的 MessageConverter 配置](http://ibear.me/2016/02/15/170)




