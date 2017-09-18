---
title: Thymeleaf 集成
date: 2017-08-06 09:42:22
tags: SpringWeb
---

**有了 Freemarker，Velocity 等模版后，为什么要选择 Thymeleaf？**

* Freemarker 的模版还是有一些非 HTML 的标签在里面，对于前端来说需要学习相关语法
* Velocity 虽然也很好，但是已经很久不更新了，Spring 5 已经官方宣布不支持 Velocity 了
* Thymeleaf 的语法就是 HTML 的语法，动态内容部分使用 HTML 的属性来实现，属性部分不会影响 HTML 的设计，**前后端可以很好的分离**<!--more-->

## Gradle 依赖

```groovy
compile ('org.thymeleaf:thymeleaf:3.0.7.RELEASE')
compile ('org.thymeleaf:thymeleaf-spring4:3.0.7.RELEASE')
```

## 在 `spring-mvc.xml` 里添加 Thymeleaf 的配置

```xml
<!-- 使用 thymeleaf 模版 -->
<bean id="templateResolver" class="org.thymeleaf.spring4.templateresolver.SpringResourceTemplateResolver">
    <property name="prefix" value="/WEB-INF/view/" />
    <property name="templateMode" value="HTML5" />
    <property name="cacheable" value="false" /> <!-- cacheable 线上环境用 true，开发环境用 false -->
</bean>

<bean id="templateEngine" class="org.thymeleaf.spring4.SpringTemplateEngine">
    <property name="templateResolver" ref="templateResolver" />
</bean>

<bean class="org.thymeleaf.spring4.view.ThymeleafViewResolver">
    <property name="templateEngine" ref="templateEngine" />
    <property name="characterEncoding" value="UTF-8" />  <!--解决中文乱码-->
</bean>
```

## 模版中使用 Model 的数据

```html
<body>
    <span th:text="'Welcome ' + ${name}">默认值</span>
    <span th:text="|Welcome ${name}|">默认值</span>

    <ul>
        <li th:each="animal : ${animals}" th:text="${animal}">小动物 - 此处文本会被覆盖</li>
    </ul>

    <a th:href="@{/login}"  th:if="${session.user == null}">Login</a>
    <a th:href="@{/logout}" th:if="${session.user != null}">Logout</a>
</body>
```

```java
@GetMapping("/leaf")
public String hello(ModelMap model) {
    List<String> animals = new LinkedList<>();
    animals.add("Tiger");
    animals.add("Cat");
    animals.add("Lion");
    model.put("animals", animals);
    model.put("name", "道格拉斯·狗");
  
    return "leaf.html";
}
```

