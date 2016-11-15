---
title: SpringMvc 接收日期参数
date: 2016-11-15 19:49:22
tags: Spring-Mvc
---
SpringMvc 的请求中的参数默认(字符串)是不能自动地转换为日期的，需要使用 Converter, InitBinder 或者 Formatter 来把请求中的参数转换为日期。

<!--more-->

## 使用 Converter
1. 定义字符串转换为日期的类 DateConverter

    ```java
    package com.xtuer.converter;

    import org.springframework.core.convert.converter.Converter;
    
    import java.text.ParseException;
    import java.text.SimpleDateFormat;
    import java.util.Date;
    
    public class DateConverter implements Converter<String, Date> {
        @Override
        public Date convert(String source) {
            String pattern = source.length()==10 ? "yyyy-MM-dd" : "yyyy-MM-dd HH:mm:ss";
            SimpleDateFormat format = new SimpleDateFormat(pattern);
    
            try {
                return format.parse(source);
            } catch (ParseException e) {
                e.printStackTrace();
            }
    
            return null;
        }
    }
    ```
2. SpringMvc 的配置文件中注册 Converter

    ```xml
    <mvc:annotation-driven conversion-service="customConversionService">
        <mvc:message-converters register-defaults="true">
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                <constructor-arg value="UTF-8" />
            </bean>
            <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
                <property name="objectMapper">
                    <bean class="com.fasterxml.jackson.databind.ObjectMapper">
                        <property name="dateFormat">
                            <bean class="java.text.SimpleDateFormat">
                                <constructor-arg type="java.lang.String" value="yyyy-MM-dd HH:mm:ss"/>
                            </bean>
                        </property>
                    </bean>
                </property>
            </bean>
        </mvc:message-converters>
        <mvc:argument-resolvers>
            <bean class="org.springframework.mobile.device.DeviceWebArgumentResolver"/>
        </mvc:argument-resolvers>
    </mvc:annotation-driven>

    <bean id="customConversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <bean class="com.xtuer.converter.DateConverter"/>
            </set>
        </property>
    </bean>
    ```

3. 处理请求

    ```
    @GetMapping("/to-date")
    @ResponseBody
    public Date toDate(Date date) {
        System.out.println("=========>" + date);
        return date;
    }
    ```

4. 访问 <http://localhost:8080/to-date?date=2016-04-12> 或者 <http://localhost:8080/to-date?date=2016-04-12%2012:12:12> 能得到日期 `2016-04-12` 和 `2016-04-12 12:12:12`

> Converter 是全局的，可以在所有 Controller 中使用。

## 使用 InitBinder
1. Controller 中定义 InitBinder

    ```java
    @InitBinder("date")
    public void initDate(WebDataBinder binder) {
        binder.registerCustomEditor(Date.class, new CustomDateEditor(new SimpleDateFormat("yyyy-MM-dd"), true));
    }
    ```

2. 处理请求

    ```java
    @GetMapping("/to-date")
    @ResponseBody
    public Date toDate(Date date) {
        System.out.println("=========>" + date);
        return date;
    }
    ```

> 注意: @InitBinder("date") 中 date 必须和 toDate(Date date) 中的 date 名字一样，当然，请求的参数中也必须有名为 date 的参数。
> 
> @InitBinder 只能在当前 Controller 中使用，当有多个地方都需要把参数转换为日期对象，则使用 Converter 更适合。
