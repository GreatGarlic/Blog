---
title: Freemarker 集成
date: 2016-10-15 11:43:26
tags: Spring-Web
---
JSP 和 `Freemarker` 都用于显示层，但是都有自己的优缺点。

##### Freemarker 的优点：
1. 不能编写 Java 代码，可以实现`严格的 MVC 分离`，可维护性好
2. 性能不错，比 JSP 快
3. 对 JSP 标签支持良好
4. 内置大量常用函数
5. 宏定义非常简单(类似 JSP 标签)
6. 使用表达式语言
7. 美工和技术的工作分离（例如命名为 .htm 的格式，不需要经过 Server 就能在浏览器里看到效果，JSP 这一点不太方便）

##### Freemarker 的缺点：  
1. 不是官方标准
2. 用户群体和第三方标签库没有 JSP 多

<!--more-->

## Gradle 依赖
```groovy
compile 'org.freemarker:freemarker:2.3.23'
```

## 在 `spring-mvc.xml` 里添加 Freemarker 的配置
> Order 值越小，查找 View 的优先级越高，如果找不到对应的 View，接着尝试下一个 order 值更高的视图解析器查找 View

```xml
<!-- Freemarker 视图解析器 -->
<bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
    <property name="prefix" value=""/>
    <property name="order"  value="0"/>
    <property name="cache"  value="true"/>
    <property name="contentType" value="text/html; charset=UTF-8"/>
</bean>

<!-- Freemarker 文件放在目录 WEB-INF/view/ftl 下 -->
<bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
    <property name="templateLoaderPath" value="/WEB-INF/view/fm/"/>
    <property name="freemarkerSettings">
        <props>
            <prop key="defaultEncoding">UTF-8</prop>
        </props>
    </property>
</bean>
```

## 模版中使用 Model 的数据
`${variableName}`

## 模版里使用 contextPath
为了在模版里使用 HttpServletRequest，需要设置 `requestContextAttribute`

```xml
<bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
    <property name="prefix" value=""/>
    <property name="order"  value="0"/>
    <property name="cache"  value="true"/>
    <property name="contentType" value="text/html; charset=UTF-8"/>
    <property name="requestContextAttribute" value="request"/>
</bean>
```
    
在模版里使用 Request: `${request.contextPath}`

## spring-mvc.xml 里定义 Freemarker 的变量
需要配置 `freemarkerVariables`

```xml
<!-- Freemarker 文件放在目录 WEB-INF/view/fm 下 -->
<bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
    <property name="templateLoaderPath" value="/WEB-INF/view/fm/"/>
    <property name="freemarkerSettings">
        <props>
            <prop key="defaultEncoding">UTF-8</prop>
        </props>
    </property>

    <!-- 定义变量, 在模版里直接可以使用 -->
    <property name="freemarkerVariables">
        <map>
            <entry key="static" value=""/>
        </map>
    </property>
</bean>
```

在模版里使用变量: `${static}`

