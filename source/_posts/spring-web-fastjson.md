---
title: Fastjson 简单使用
date: 2016-10-16 08:09:09
tags: [Spring-Web, Util]
---
Fastjson API 入口类是 com.alibaba.fastjson.JSON，常用的序列化操作都可以在 JSON 类上的静态方法直接完成。

<!--more-->

## Gradle 依赖
```groovy
compile 'com.alibaba:fastjson:1.2.17'
```

## 主要 API
```java
// 把 JSON 文本 parse 为 JSONObject 或者 JSONArray 
public static final Object parse(String text); 

// 把 JSON 文本 parse 成 JSONObject  
public static final JSONObject parseObject(String text)；   

// 把 JSON 文本 parse 为 JavaBean
public static final <T> T parseObject(String text, Class<T> clazz); 

// 把 JSON 文本 parse 成 JSONArray 
public static final JSONArray parseArray(String text);  

// 把 JSON 文本 parse 成 JavaBean 集合 
public static final <T> List<T> parseArray(String text, Class<T> clazz); 

// 将 JavaBean 序列化为 JSON 文本 
public static final String toJSONString(Object object); 

// 将 JavaBean 序列化为带格式的 JSON 文本 
public static final String toJSONString(Object object, boolean prettyFormat);

// 将 JavaBean 转换为 JSONObject 或者 JSONArray
public static final Object toJSON(Object javaObject); 
```

> `SerializeWriter`: 相当于 `StringBuffer`  
> `JSONArray`: 相当于 `List<Object>`  
>  `JSONObject`: 相当于 `Map<String, Object>`

## 测试
```java
Demo d = new Demo();

String json = JSON.toJSONString(d);
d = JSON.parseObject(json, Demo.class);
```
