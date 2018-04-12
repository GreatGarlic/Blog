---
title: Fastjson 简单使用
date: 2016-10-16 08:09:09
tags: [SpringWeb, Util]
---
Fastjson API 入口类是 com.alibaba.fastjson.JSON，常用的序列化操作都可以在 JSON 类上的静态方法直接完成。

<!--more-->

## Gradle 依赖
```groovy
compile 'com.alibaba:fastjson:1.2.41'
```

## 主要 API

把对象转换为 JSON 字符串

```java
public static String toJSONString(Object object) // 转为压缩格式的，去掉多余的空格，占用空间少
public static String toJSONString(Object object, boolean prettyFormat) // prettyFormat 为 true 转为格式化后的，可读性好
```

把 JSON 字符串转换为对象

```java
public static <T> T parseObject(String text, Class<T> clazz);
public static <T> T parseObject(String text, TypeReference<T> type, Feature... features);
```

转为字符串时忽略某一个属性，使用 ignores

```java
@JSONType(ignores = {"children"})
public static class Node {
    Long id;
    Long parentId;

    List<Node> children = new LinkedList<>();
}
```

前端 JS 不支持 Long，可以把 Long 转换为 String 后返回给前端

```java
1. 在后台将这个Long类型的字段转换成String类型的，风险比较大
2. 使用 Fastjson 的提供的注解: 
   @JSONField(serializeUsing = ToStringSerializer.class)
   private Long id
3. 使用浏览器兼容模式，全局配置 BrowserCompatible 的 serializerFeatures，例如 SpringMVC 里
   <property name="fastJsonConfig">
       <bean class="com.alibaba.fastjson.support.config.FastJsonConfig">
           <property name="serializerFeatures">
               <list>
                   <value>BrowserCompatible</value> <!-- 解决 JS 不支持 Long 类型: Long 输出为字符串 -->
               </list>
           </property>
       </bean>
   </property>
   注意: 这种方式中文会转为 UTF-8 的编码，如 {"id":2,"info":"\u6D77\u9F99"}，不过浏览器里能自动识别
4. 自定义 FastJsonConfig，只把 Long 转为字符串，中文进行 UTF-8 编码，比方法 3 好一些，请参考 http://sparkgis.com/java/2018/02/springmvc使用fastjson并解决长数值精度丢失问题-原-springmvc使用fastjson并/
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
> `JSONObject`: 相当于 `Map<String, Object>`

## 参考资料

* [Wiki](https://github.com/Alibaba/fastjson/wiki/首页)
* [常见问题](https://github.com/alibaba/fastjson/wiki/常见问题): 例如怎么处理日期

