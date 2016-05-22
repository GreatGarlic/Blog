---
title: Commons-Lang3 例子
date: 2016-05-22 14:01:33
tags: [Java, Util]
---

可以使用 Apache Commons Lang3:

* 日期格式化
* 序列化和反序列化
* 生成随机字符串
* 字符串 join, 包含, 空判断, 缩写, 补全输出指定长度的字符串 (leftPad, center, rightPad) 等
* 获取系统信息
* 获取 Class 的信息
* HTML escape and unescape
* 数字相关, 如求数组中的最大最小值

<!--more-->

## 依赖
```groovy
compile 'org.apache.commons:commons-lang3:3.4'
```

## 日期格式化
```java
    public void dateFormatUtilsDemo() {
        System.out.println(genHeader("DateFormatUtilsDemo"));
        System.out.println("格式化日期输出.");
        System.out.println(DateFormatUtils.format(System.currentTimeMillis(), "yyyy-MM-dd HH:mm:ss"));
    }
```

## 序列化和反序列化
```java
    public void serializationUtilsDemo() {
        String src = "道格拉斯二狗";
        byte[] result = SerializationUtils.serialize(src); // 序列化

        String dst = SerializationUtils.deserialize(result); // 反序列化
        System.out.println(dst); // 输出: 道格拉斯二狗
    }
```

## 生成随机字符串
```java
    public void randomStringUtilsDemo() {
        System.out.println("**RandomStringUtilsDemo**");
        System.out.println("生成指定长度的随机字符串,好像没什么用.");
        System.out.println(RandomStringUtils.random(500));

        System.out.println("在指定字符串中生成长度为 n 的随机字符串.");
        System.out.println(RandomStringUtils.random(5, "abcdefghijk"));

        System.out.println("指定从字符或数字中生成随机字符串.");
        System.out.println(RandomStringUtils.random(5, true, false));
        System.out.println(RandomStringUtils.random(5, false, true));
    }
```

## 字符串 join, 包含, 空判断, 缩写等
```java
    public void stringUtilsDemo() {
        System.out.println("**StringUtilsDemo**");
        System.out.println("将字符串重复n次，将文字按某宽度居中，将字符串数组用某字符串连接.");
        String[] header = new String[3];
        header[0] = StringUtils.repeat("*", 50);
        header[1] = StringUtils.center("  StringUtilsDemo  ", 50, "^O^");
        header[2] = header[0];
        String head = StringUtils.join(header, "/n");
        System.out.println(head);

        System.out.println("缩短到某长度,用...结尾.");
        System.out.println(StringUtils.abbreviate("The quick brown fox jumps over the lazy dog.", 10));
        System.out.println(StringUtils.abbreviate("The quick brown fox jumps over the lazy dog.", 15, 10));

        System.out.println("返回两字符串不同处索引号.");
        System.out.println(StringUtils.indexOfDifference("aaabc", "aaacc"));

        System.out.println("返回两字符串不同处开始至结束.");
        System.out.println(StringUtils.difference("aaabcde", "aaaccde"));

        System.out.println("检查一字符串是否为另一字符串的子集.");
        System.out.println(StringUtils.containsOnly("aad", "aadd"));

        System.out.println("检查一字符串是否不是另一字符串的子集.");
        System.out.println(StringUtils.containsNone("defg", "aadd"));

        System.out.println("检查一字符串是否包含另一字符串.");
        System.out.println(StringUtils.contains("defg", "ef"));
        System.out.println(StringUtils.containsOnly("ef", "defg"));

        System.out.println("返回可以处理null的toString().");
        System.out.println(StringUtils.defaultString("aaaa"));
        System.out.println("?" + StringUtils.defaultString(null) + "!");

        System.out.println("去除字符中的空格.");
        System.out.println(StringUtils.deleteWhitespace("aa  bb  cc"));

        System.out.println("判断是否是某类字符.");
        System.out.println(StringUtils.isAlpha("ab"));
        System.out.println(StringUtils.isAlphanumeric("12"));
        System.out.println(StringUtils.isBlank(""));
        System.out.println(StringUtils.isNumeric("123"));
        
        System.out.println("左边加字符, 总长度为指定的长度 20, 格式化输出时用");
        System.out.println(StringUtils.leftPad("道格拉斯二狗", 20, " "));
    }
```

## 获取系统信息
```java
    public void systemUtilsDemo() {
        System.out.println(genHeader("SystemUtilsDemo"));
        System.out.println("获得系统文件分隔符.");
        System.out.println(SystemUtils.FILE_SEPARATOR);

        System.out.println("获得源文件编码.");
        System.out.println(SystemUtils.FILE_ENCODING);

        System.out.println("获得ext目录.");
        System.out.println(SystemUtils.JAVA_EXT_DIRS);

        System.out.println("获得java版本.");
        System.out.println(SystemUtils.JAVA_VM_VERSION);

        System.out.println("获得java厂商.");
        System.out.println(SystemUtils.JAVA_VENDOR);
    }
```

## 获取 Class 的信息
```java
    public void classUtilsDemo() {
        System.out.println(genHeader("ClassUtilsDemo"));
        System.out.println("获取类实现的所有接口.");
        System.out.println(ClassUtils.getAllInterfaces(Date.class));

        System.out.println("获取类所有父类.");
        System.out.println(ClassUtils.getAllSuperclasses(Date.class));

        System.out.println("获取简单类名.");
        System.out.println(ClassUtils.getShortClassName(Date.class));

        System.out.println("获取包名.");
        System.out.println(ClassUtils.getPackageName(Date.class));

        System.out.println("判断是否可以转型.");
        System.out.println(ClassUtils.isAssignable(Date.class, Object.class));
        System.out.println(ClassUtils.isAssignable(Object.class, Date.class));
    }
```

## HTML escape and unescape
```java
    public void stringEscapeUtilsDemo() {
        System.out.println(genHeader("StringEscapeUtils"));
        System.out.println("转换特殊字符.");
        System.out.println("html:" + StringEscapeUtils.escapeHtml4("/n\n"));
        System.out.println("html:" + StringEscapeUtils.unescapeHtml4("<p>"));
    }
```

## 数字相关, 如求数组中的最大最小值
```java
    public void numberUtils() {
        System.out.println(genHeader("NumberUtils"));
        System.out.println("字符串转为数字(不知道有什么用).");
        System.out.println(NumberUtils.toInt("ba", 33));

        System.out.println("从数组中选出最大值.");
        System.out.println(NumberUtils.max(new int[] { 1, 2, 3, 4 }));

        System.out.println("判断字符串是否全是整数.");
        System.out.println(NumberUtils.isDigits("123.1"));

        System.out.println("判断字符串是否是有效数字.");
        System.out.println(NumberUtils.isNumber("0123.1"));
    }
```

## 主程序
```java
import org.apache.commons.lang3.*;
import org.apache.commons.lang3.math.NumberUtils;
import org.apache.commons.lang3.time.DateFormatUtils;

import java.util.Date;

public class LangDemo {
    public static void main(String[] args) {
        LangDemo langDemo = new LangDemo();

        langDemo.serializationUtilsDemo();
        langDemo.randomStringUtilsDemo();
        langDemo.stringUtilsDemo();
        langDemo.systemUtilsDemo();
        langDemo.classUtilsDemo();
        langDemo.stringEscapeUtilsDemo();
        langDemo.numberUtils();
        langDemo.dateFormatUtilsDemo();
    }

    private String genHeader(String head) {
        String[] header = new String[3];
        header[0] = StringUtils.repeat("*", 50);
        header[1] = StringUtils.center("  " + head + "  ", 50, "^O^");
        header[2] = header[0];
        return StringUtils.join(header, "");
    }
}
```

## 参考
[Commons Lang3 功能示例](http://blog.csdn.net/djun100/article/details/11879797)
