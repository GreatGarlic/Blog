---
title: Fastjson 简单使用
date: 2016-10-16 08:09:09
tags: [SpringWeb, Util]
---
Fastjson API 入口类是 com.alibaba.fastjson.JSON，常用的序列化操作都可以在 JSON 类上的静态方法直接完成。

<!--more-->

## Gradle 依赖
```groovy
compile 'com.alibaba:fastjson:1.2.17'
```

## 主要 API
把对象转换为 JSON 字符串

```java
public static String toJSONString(Object object);
```

把 JSON 字符串转换为对象

```java
public static <T> T parseObject(String text, Class<T> clazz);
public static <T> T parseObject(String text, TypeReference<T> type, Feature... features);
```

## TypeReference
什么时候用 `TypeReference` 呢？使用 `parseObject()` 转换的结果使用范型时。

> **public class TypeReference<T> extends Object**  
>
> Represents a generic type T. Java doesn't yet provide a way to represent generic types(例如 `List<String>.class` 是不存在的), so this class does. Forces clients to create a subclass of this class which enables retrieval the type information even at runtime.
> For example, to create a type literal for `List<String>`, you can create an empty anonymous inner class:
>
> &nbsp;&nbsp;&nbsp;&nbsp;`TypeReference<List<String>> list = new TypeReference<List<String>>() {};`
>
> This syntax cannot be used to create type literals that have wildcard parameters, such as `Class<?>` or `List<? extends CharSequence>`.

## 测试
```java
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.TypeReference;

import java.util.LinkedList;
import java.util.List;

public class TestFastJson {
    public static void main(String[] args) {
        Box box = new Box("Foo");

        String json = JSON.toJSONString(box);
        System.out.println(json); // {"data":"Foo"}

        box = JSON.parseObject(json, Box.class);
        System.out.println(box); // Box{data=Foo}

        List<Box> list = new LinkedList<>();
        list.add(new Box("Alice"));
        list.add(new Box("John"));
        json = JSON.toJSONString(list);
        System.out.println(json); // [{"data":"Alice"},{"data":"John"}]

        list = JSON.parseObject(json, List.class);
        System.out.println(list); // [{"data":"Alice"}, {"data":"John"}]，可以看到，输出的不是 Box.toString() 输出的内容，说明 list 中存储的不是 Box
        // System.out.println(list.get(0).getClass()); // Error: ClassCastException: com.alibaba.fastjson.JSONObject cannot be cast to Box

        list = JSON.parseObject(json, new TypeReference<List<Box>>() {}); // 使用 parseObject() 转换的结果使用范型时使用 TypeReference
        System.out.println(list.get(0).getClass()); // class Box
        System.out.println(list); // [Box{data=Alice}, Box{data=John}]
    }
}

class Box {
    private String data;

    public Box() {

    }

    public Box(String data) {
        this.data = data;
    }

    public String getData() {
        return data;
    }

    public void setData(String data) {
        this.data = data;
    }

    @Override
    public String toString() {
        return String.format("Box{data=%s}", data);
    }
}
```

## 常用 API
```java
​```java
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

## 参考资料

* [Wiki](https://github.com/Alibaba/fastjson/wiki/首页)
* [常见问题](https://github.com/alibaba/fastjson/wiki/常见问题): 例如怎么处理日期

