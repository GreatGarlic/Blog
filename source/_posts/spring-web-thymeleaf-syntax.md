---
title: Thymeleaf 语法
date: 2017-08-06 10:07:37
tags: Spring-Web
---

Thymeleaf 使用 HTML 元素的属性获取 model 中的数据，属性的前缀是 `th:`，例如 `th:text`, `th:src`。

## 变量访问

```html
<span th:text="${name}">Thymeleaf 解析后会被覆盖</span>
<span th:text="|Welcome ${name}|">Thymeleaf 解析后会被覆盖</span>
<span th:text="'Welcome ' + ${name}">Thymeleaf 解析后会被覆盖</span>
```

字符串拼接时 `|...|` 的方式更简洁，但是里面不能包含表达式，第三种方式功能强大，可以包含表达式。

变量访问也可以使用级联的方式: `${user.name}`。

## 使用 URL

```html
<a th:href="@{/login}" th:if="${session.user == null}">Login</a>
```

使用 `th:href="@{/uri}"` 引入 URL，`/` 开头时会在 URI 前面加上项目的 **context path**。<!--more-->

## 运算符

```html
<!-- model 中 age 为 30 -->
<span th:text="|${age} 大于 20|" th:if="${age} > 20">显示</span>
<span th:text="|${age} 小于 20|" th:if="${age} < 20">不显示</span>
<span th:text="4%2==0 ? '偶数' : '素数'"></span>
<span th:text="3%2==0 ? '偶数' : '素数'"></span>
```

逻辑运算符 `>`、 `<`、`<=`、`>=`、`==`、`!=`、`三元表达式` 等都可以使用

## if else

```html
<body>
    <div th:if="${age} > 20">
        大于 20
    </div>
    <div th:unless="${age} > 20">
        小于 20
    </div>
</body>
```

使用 `if unless` 实现 **if else** 的功能，unless 是除了的意思。

Thymeleaf 里没有啥好办法实现 `if, else if, eles` 的功能。

## switch

```html
<div th:switch="${role}">
    <p th:case="user">User</p>
    <p th:case="${admin}">Admin</p>
    <p th:case="*">Unknown</p>
</div>
```

> 默认属性 default 可以用 `*` 表示

## 循环

```html
<ul>
    <li th:each="animal : ${animals}" th:text="${animal}">小动物 - 此处文本会被覆盖</li>
</ul>
```

循环使用 `th:each`，上面会重复生成多个 **li**。

## 格式化时间

```html
<!-- Controller 中 model.put("date", new Date()); -->
<p th:text="${#dates.format(date, 'yyyy-MM-dd HH:mm:ss')}"></p>
```

## Utility

为了模板更加易用，Thymeleaf 内置了一些 utility 函数，可以通过 `#` 直接访问，例如

* [dates](http://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf.html#dates)
* [strings](http://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf.html#strings)
* [numbers](http://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf.html#numbers)
* 更多的请参考[帮助文档](http://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf.html#numbers)

```html
${#dates.setFormat(datesSet)}
${#dates.format(date)}
${#dates.format(date, 'dd/MMM/yyyy HH:mm')}

${#strings.isEmpty(name)}
${#strings.contains(name,'ez')}
${#strings.startsWith(name,'Don')} 
${#strings.trim(str)}

${#numbers.formatInteger(num,3)}
${#numbers.formatDecimal(num,3,2)}
${#numbers.setFormatInteger(numSet,3,'POINT')}
${#numbers.setFormatDecimal(numSet,3,2)}
${#numbers.setFormatDecimal(numSet,3,2,'COMMA')}

...
```



## 参考资料

* [Thymeleaf 官方文档](http://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf.html#introducing-thymeleaf)