---
title: Java Code Snippets
date: 2016-06-02 13:22:45
tags: [Util, Java]
---

一些 Java 里常用的代码片段，例如 Bean to String 等。

<!--more-->

## Bean to String
使用 Apache Commons Lang3

```java
import org.apache.commons.lang3.builder.ReflectionToStringBuilder;
import org.apache.commons.lang3.builder.ToStringStyle;
import org.apache.commons.lang3.builder.ToStringBuilder;

System.out.println(ReflectionToStringBuilder.toString(obj));
System.out.println(ReflectionToStringBuilder.toString(obj, ToStringStyle.MULTI_LINE_STYLE));
System.out.println(ToStringBuilder.reflectionToString(p));
```

使用 Jackson，Json 的格式更漂亮

```java
import com.fasterxml.jackson.databind.ObjectMapper;

ObjectMapper mapper = new ObjectMapper();
System.out.println(mapper.writeValueAsString(obj));
```

## 取得当前的方法名
```java
Thread.currentThread().getStackTrace()[1].getMethodName()
```

## 依赖
* Apache Commons Lang3: `org.apache.commons:commons-lang3:3.4`
* Jackson: `com.fasterxml.jackson.core:jackson-databind:2.7.3`
