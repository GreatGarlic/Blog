---
title: View Controller
date: 2016-10-15 15:35:16
tags: SpringWeb
---
有很多静态页面，里没有动态的内容，如果写 Controller 去做映射的话又感觉很麻烦，都是体力活，没什么意思，这时可以用 `mvc:view-controller` 进行映射达到相同的效果而又不需要写 Controller。

> 因为这个配置需要 Controller 的支持，所以 view 的文件需要放在模版所在文件夹。

<!--more-->

```xml
<!-- rest-view.html 是 View 的名字 -->
<mvc:view-controller path="/rest-view" view-name="rest-view.html"/>
```

访问 <http://localhost:8080/rest-view>，则 View Resolver 访问的是 `/WEB-INF/view/fm/rest-view.html`

## mvc:view-controller 配置
例如存放在文件 `spring-view-controller.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc.xsd">
    <mvc:view-controller path="/rest-view" view-name="rest-view.html"/>
</beans>
```

## 引入 mvc:view-controller 配置
在 spring-mvc.xml 中引入 spring-view-controller.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc.xsd">
    <import resource="classpath:config/spring-view-controller.xml"/>
    <!-- 其他部分 -->
</beans>
```
