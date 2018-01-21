---
title: Thymeleaf 语法
date: 2017-08-06 10:07:37
tags: SpringWeb
---

Thymeleaf 使用 HTML 元素的属性获取 model 中的数据，属性的前缀是 `th:`，例如 `th:text`, `th:src`。

## 变量访问

```html
<!-- 使用属性的方式访问变量 -->
<span th:text="${name}">Thymeleaf 解析后会被覆盖</span>
<span th:text="|Welcome ${name}|">Thymeleaf 解析后会被覆盖</span>
<span th:text="'Welcome ' + ${name}">Thymeleaf 解析后会被覆盖</span>

<!-- 非属性的方式访问变量 -->
<span>[[${name}]]</span>
```

字符串拼接时 `|...|` 的方式更简洁，但是里面不能包含表达式，第三种方式功能强大，可以包含表达式。

变量访问也可以使用级联的方式: `${user.name}`。

## null 处理

表达式 `${foo}?` 当 foo 为 null 时返回 false；级联调用时变量访问前先用 `?` 进行判断可以减少一个一个的条件判断

```html
<div th:text="${foo}?'Alt'"></div>
<div th:text="${foo?.bar?.fix}"></div>
```

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

## 常用 th

* th:attr: 可以同时设置很多属性，也可以设置自定义属性，偷懒的时候就不用记忆很多 `th:` 开头的属性了

  ```html
  <p th:attr="src=@{/images/gtvglogo.png}, title=${logo}, alt=${logo}"></p>
  <p th:attr="data-level=${level}, title=${level}" th:text="${level}"></p>
  ```

* th:text

* th:utext

* th:name

* th:value

* th:class

* th:href

* th:src

* th:alt

* th:title

* th:field

* Appending and prepending

  * th:attrappend

  * th:attrprepend

    ```html
    <body>
        <!-- 以下表达式效果都一样 -->
        <p class="foo" th:attrappend="class=${' '+level}">Hello</p>
        <p class="foo" th:attrappend="class=' '+${level}">Hello</p>
        <p class="foo" th:attrappend="class=| ${level}|">Hello</p>
    </body>
    ```

* th:checked

* th:selected

* th:object

* 具体请参考 [Setting value to specific attributes](http://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf.html#setting-value-to-specific-attributes)

## 参考资料

* [Thymeleaf 官方文档](http://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf.html#introducing-thymeleaf)
* [Thymeleaf 3 ten-minute migration guide](http://www.thymeleaf.org/doc/articles/thymeleaf3migration.html)

