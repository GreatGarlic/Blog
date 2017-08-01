---
title: Freemarker 语法
date: 2016-10-15 14:47:10
tags: Spring-Web
---
FreeMarker 是一个用 Java 语言编写的模板引擎，它基于模板来生成文本输出。FreeMarker 与 Web 容器无关，即在 Web 运行时，它并不知道 Servlet 或 HTTP。它不仅可以用作表现层的实现技术，而且还可以用于生成 XML，JSP 或 Java 等。

<!--more-->

## FreeMarker模板文件主要由如下4个部分组成
* `文本`：直接输出的部分 
* `注释`：<#-- ... --> 格式部分，不会输出 
* `插值`：即 ${...} 或 #{...} 格式的部分，将使用数据模型中的部分替代输出 
* `指令`：FreeMarker 指定，和 HTML 标记类似，名字前加 # 予以区分，不会输出

```html
<html>
<head>
    <title>Welcome!</title>
</head>
<body>
    <#-- 注释部分 -->
    <#-- 下面使用插值 -->
    <h1>Welcome ${username} !</h1>

    <p>We have these animals:</p>
    <u1>
        <!-- 使用FTL指令 -->
        <#list animals as animal>
        <li>${animal.name} for ${animal.price} Euros</li>
        </#list>
    </u1>
</body>
</html
```

## 控制语句
```html
<#if condition> 
    ... 
<#elseif condition2> 
    ... 
<#elseif condition3> 
    ... 
<#else>
    ...
</#if>

<#if (age>60)>老年人  
<#elseif (age>40)>中年人  
<#elseif (age gt 20)>青年人  
<#else> 少年人  
</#if> 
  
<#if list.type==0><td>全部频道</td>
<#elseif list.type==1><td>抚州一套</td>
<#elseif list.type==2><td>抚州二套</td>
</#if>

<#if str == "success">
    xxx
</#if>
```

```html
<#switch value> 
    <#case refValue1> 
        ... 
        <#break> 
    <#case refValue2> 
        ... 
        <#break> 
    <#case refValueN> 
        ... 
        <#break> 
    <#default> 
        ... 
</#switch>
```

## 通用插值
1. 插值结果为字符串值：直接输出表达式结果 
2. 插值结果为数字值：根据默认格式（由#setting指令设置）将表达式结果转换成文本输出。可以使用内建的字符
3. 插值结果为日期值：根据默认格式(由#setting指令设置)将表达式结果转换成文本输出
4. 数字格式化插值

```html
<#settion number_format="currency"/> 
<#assign answer=42/> 
${answer} 
${answer?string} <#-- the same as ${answer} --> 
${answer?string.number} 
${answer?string.currency} 
${answer?string.percent} 
${answer} 
```

输出结果是: 
> $42.00 
> $42.00 
> 42 
> $42.00 
> 4,200%

---

```html
${lastUpdated?string("yyyy-MM-dd HH:mm:ss zzzz")} 
${lastUpdated?string("EEE, MMM d, ''yy")} 
${lastUpdated?string("EEEE, MMMM dd, yyyy, hh:mm:ss a '('zzz')'")}
```

输出结果是: 
> 2008-04-08 08:08:08 Pacific Daylight Time 
> Tue, Apr 8, '03 
> Tuesday, April 08, 2003, 08:08:08 PM (PDT) 

---

```html
数字格式化插值可采用 #{expr;format} 形式来格式化数字，其中format可以是：
mX:小数部分最小X位 
MX:小数部分最大X位 

如下面的例子: 
<#assign x=2.582/> 
<#assign y=4/> 
#{x; M2} <#-- 输出2.58 --> 
#{y; M2} <#-- 输出4 --> 
#{x; m2} <#-- 输出2.6 --> 
#{y; m2} <#-- 输出4.0 --> 
#{x; m1M2} <#-- 输出2.58 --> 
#{x; m1M2} <#-- 输出4.0 -->
```

## 函数调用
函数调用在变量后用 `?`：`${username?upper_case}`

## 指令
指令调用用 `<#commandName arguments></#commandName>`

## 宏（相当于 JSP 的 tag 定义）
```html
    <!-- 定义宏用 <#macro macroName parameterList> -->
    <#macro list title items>
        <p>${title?cap_first}:
        <ul>
            <#list items as item>
            <li>${item}
            </#list>
        </ul>
    </#macro>
    
    <!-- 调用宏用 <@macroName arguments> -->
    <@list items=animals title="animals"/>
```

## include 指令 
include 指令的作用类似于 JSP 的包含指令，用于包含指定页。include 指令的语法格式如下：
`<#include filename [options]>`

## import 
`<#import path as hash>` 
类似于 java 里的 import，它导入文件，然后就可以在当前文件里使用被导入文件里的宏组件。

假设 mylib.ftl 里定义了`宏 copyright` 那么我们在其他模板页面里可以这样使用 

```html
<#import "/libs/mylib.ftl" as my> 
<@my.copyright date="1999-2002"/> 
```

`my` 在 freemarker 里被称作 namespace

## noparse 指令
noparse 指令指定 FreeMarker 不处理该指定里包含的内容，该指令的语法格式如下：
`<#noparse>...</#noparse>` 

```html
    <#noparse>
        <#list animals as animal>
        <li>${animal.name} for ${animal.price} Euros</li>
        </#list>
    </#noparse>
```

## 转义 HTML 特殊字符
`${username?html}`

## 防止空指针报错
变量名后用 `!` 加默认值：`${foo!"Default"}`，如果 foo 为 null 则输出 Default

## 示例

```html
<!-- UUID: 0D85193C-E51F-4A06-9C65-DB1E5AC999ED -->
<html>
<head>
    <title>Welcome!</title>
</head>
<body>
    <#-- 注释部分 -->

    <#-- 下面使用插值 -->
    <h1>Welcome ${username}!</h1>

    如果直接访问为 null 则报错，防止 null 报错可以在 ! 后跟默认值: ${foo!"Default"}<br>
    函数调用用问号后跟函数名: ${username?upper_case}<br>
    ${username?html}

    <p>访问 list，指令中的参数不加 $ {}: <br>
    We have these animals:</p>
    <u1>
        <!-- 使用FTL指令 -->
        <#list animals as animal>
        <li>${animal.name} for ${animal.price} Euros</li>
        </#list>
    </u1>

    定义循环输出的宏: list 是宏的名字，title, items 是宏的参数<br>
    <#macro list title items>
        <p>${title?cap_first}:
        <ul>
            <#list items as item>
            <li>${item}
            </#list>
        </ul>
    </#macro>

    调用宏用：
    <@list items=animals title="animals"/>

    <#noparse>
        <#list animals as animal>
        <li>${animal.name} for ${animal.price} Euros</li>
        </#list>
    </#noparse>
</body>
</html>
```

---

`domain.Animal`

```java
package domain;

public class Animal {
    private String name;
    private Integer price;

    public Animal(String name, Integer price) {
        this.name = name;
        this.price = price;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getPrice() {
        return price;
    }

    public void setPrice(Integer price) {
        this.price = price;
    }
}
```

---

`controller.FreemarkerController`

```java
package controller;

import domain.Animal;
import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.RequestMapping;

import java.util.LinkedList;
import java.util.List;

@Controller
public class FreemarkerController {
    @RequestMapping("/freemarker-syntax")
    public String syntax(ModelMap map) {
        map.addAttribute("username", "Alice");

        List<Animal> animals = new LinkedList<Animal>();
        animals.add(new Animal("Dog", 10));
        animals.add(new Animal("Pig", 20));
        animals.add(new Animal("Cat", 30));
        animals.add(new Animal("Tiger", 40));

        map.addAttribute("animals", animals);

        return "freemarker-syntax.fm";
    }
}
```

## 参考资料
* [Freemarker 语法参考](http://freemarker.org/docs/dgui.html)
