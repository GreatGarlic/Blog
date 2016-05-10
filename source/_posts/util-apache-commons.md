---
title: Apache Commons
date: 2016-05-10 07:17:22
tags: [Java, Util]
---

## commons-io
Class Name | Description
---------- | -----------
FilenameUtils | 文件名称一些操作，如获取文件名的后缀
FileUtils | 文件工具类，内置提供了大量文件转换方法，如 readFileToString(File,Path)
IOUtils | 主要提供了 IO 常见操作
Stream | 转换，关闭 Stream 等操作

```groovy
'commons-io:commons-io:2.5'
```

<!--more-->

## commons-lang
Class Name | Description
---------- | -----------
CharEncoding | 字符常量提供，是否是支持的字符编码判断
ArrayUtils | 数组工具类，提供数组的常用方法，null 判断等相关
StringUtils | 功能很强大，字符常见操作，isNotBlank，特色的 split 方法等
StringEscapeUtils | 对 javascript，html，sql 等语句进行过滤
SystemUtils | 系统一些常量
SerializationUtils | 序列化操作，ObjectInputStream output 等相关
NumberUtils | 数字转换字符串操作，敏捷开发，解决了开发中大量字符与 Number 转换，及异常等问题
DateUtils | 日期操作相关，如添加一天等
DateFormatUtils | 日期转换字符相关

```groovy
'org.apache.commons:commons-lang3:3.4'
```

## commons-codec
Class Name | Description
---------- | -----------
DigestUtils | MD5加密等
Base64 | 
URLCodec | 

```groovy
'commons-codec:commons-codec:1.10'
```

## commons-beanutils
Class Name | Description
---------- | -----------
ConvertUtils | 类型转换工具类，功能强大
PropertyUtils | 字段属性操作，提供了把一个 bean 转换 Map，设置获取 bean get set 方法等
BeanUtils | populate 填充，把一个 Map 转换为 Bean，与 Spring core BeanUtils 对比，略显薄弱，可用性不强
MethodUtils | 反射方法调用 invokeMethodinvokeMethod(Object object, String methodName,Object[] args) 大量重载

```groovy
'commons-beanutils:commons-beanutils:1.9.2'
```

## 参考
[Apache commons类库阅读笔记](http://my.oschina.net/lis1314/blog/672527?fromerr=xQUrw5GY)







