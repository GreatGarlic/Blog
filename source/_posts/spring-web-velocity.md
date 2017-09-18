---
title: 集成 Velocity
date: 2017-06-30 15:16:11
tags: SpringWeb
---

JSP 和 `Velocity` 都用于显示层，但是都有自己的优缺点。

> Velocity 比 Freemarker 快，而且语法也更舒服。

##### Velocity 的优点：

1. 不能编写 Java 代码，可以实现`严格的 MVC 分离`，可维护性好
2. 性能不错，比 JSP 快
3. 对 JSP 标签支持良好
4. 内置大量常用函数
5. 宏定义非常简单(类似 JSP 标签)
6. 使用表达式语言
7. 美工和技术的工作分离（例如命名为 .htm 的格式，不需要经过 Server 就能在浏览器里看到效果，JSP 这一点不太方便）

##### Velocity 的缺点：

1. 不是官方标准
2. 用户群体和第三方标签库没有 JSP 多<!--more-->

## Gradle 依赖

```groovy
compile 'org.apache.velocity:velocity:1.7'
compile 'org.apache.velocity:velocity-tools:2.0'
```

## 在 `spring-mvc.xml` 里添加 Velocity 的配置

> Order 值越小，查找 View 的优先级越高，如果找不到对应的 View，接着尝试下一个 order 值更高的视图解析器查找 View

```xml
<!-- Velocity 配置-->
<bean id="velocityConfigurer" class="org.springframework.web.servlet.view.velocity.VelocityConfigurer">
    <property name="resourceLoaderPath"  value="WEB-INF/view/"/><!-- 设置模版文件的位置 -->
    <property name="velocityProperties">
        <props>
            <prop key="directive.foreach.counter.name">loopCounter</prop>
            <prop key="directive.foreach.counter.initial.value">0</prop>
            <prop key="input.encoding">UTF-8</prop><!-- 指定模板引擎进行模板处理的编码 -->
            <prop key="output.encoding">UTF-8</prop><!-- 指定输出流的编码 -->
        </props>
    </property>
</bean>

<!-- Velocity 视图解析器 -->
<bean id="velocityViewResolver" class="org.springframework.web.servlet.view.velocity.VelocityViewResolver">
    <property name="cache" value="true"/>
    <property name="order" value="0"/> <!--值越小，越优先执行-->
    <property name="contentType" value="text/html;charset=UTF-8"/>
    <property name="exposeRequestAttributes" value="true"/><!--是否开放request属性-->
    <property name="requestContextAttribute" value="request"/><!--request属性引用名称-->

    <!--自定义变量-->
    <property name="attributes">
        <props>
            <prop key="static">http://static-cdn.com</prop>
        </props>
    </property>
</bean>
```

## 模版中使用 Model 的数据

```html
$variableName 或则 ${variableName}
```

## 模版里使用 contextPath

为了在模版里使用 HttpServletRequest，需要设置 velocityViewResolver 的 **exposeRequestAttributes** 和 **requestContextAttribute**

```xml
<property name="exposeRequestAttributes" value="true"/><!--是否开放 request 属性-->
<property name="requestContextAttribute" value="request"/><!--request 属性引用名称-->
```

在模版里使用 Request: `$request.contextPath`

## spring-mvc.xml 里定义 Velocity 的变量

需要配置 velocityViewResolver 的 `attributes` 属性

```xml
<!--自定义变量-->
<property name="attributes">
    <props>
        <prop key="static">http://static-cdn.com</prop>
    </props>
</property>
```

在模版里使用变量: `$static`

