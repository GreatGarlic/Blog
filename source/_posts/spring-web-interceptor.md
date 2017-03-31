---
title: SpringMVC 拦截器
date: 2017-03-31 17:37:59
tags: Spring-Web
---
## 1. 实现拦截器类

```java
package com.xtuer.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class MyInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) throws Exception {
        System.out.println("my interceptor");
        return true; // 返回 true 则继续处理
    }

    @Override
    public void postHandle(HttpServletRequest request,
                           HttpServletResponse response,
                           Object handler,
                           ModelAndView modelAndView) throws Exception {
        String token = UUID.randomUUID().toString().replace("-", "").toUpperCase();
        System.out.println(token);
        modelAndView.addObject("token", token);
    }

    @Override
    public void afterCompletion(HttpServletRequest request,
                                HttpServletResponse response,
                                Object handler, Exception ex) throws Exception {

    }
}
```
<!--more-->

| 函数              | 说明                                       |
| --------------- | ---------------------------------------- |
| preHandle       | Intercept the execution of a handler. Called after HandlerMapping determined an appropriate handler object, but before HandlerAdapter invokes the handler. <br>DispatcherServlet processes a handler in an execution chain, consisting of any number of interceptors, with the handler itself at the end. With this method, each interceptor can decide to abort the execution chain, typically sending a HTTP error or writing a custom response. |
| postHandle      | Intercept the execution of a handler. Called after HandlerAdapter actually invoked the handler, but before the DispatcherServlet renders the view. Can expose additional model objects to the view via the given ModelAndView. <br>DispatcherServlet processes a handler in an execution chain, consisting of any number of interceptors, with the handler itself at the end. With this method, each interceptor can post-process an execution, getting applied in inverse order of the execution chain. |
| afterCompletion | Callback after completion of request processing, that is, after rendering the view. Will be called on any outcome of handler execution, thus allows for proper resource cleanup. <br>Note: Will only be called if this interceptor's preHandle method has successfully completed and returned true! <br>As with the postHandle method, the method will be invoked on each interceptor in the chain in reverse order, so the first interceptor will be the last to be invoked. |

## 2. 配置拦截器

在 spring-mvc.xml 里配置拦截器

```xml
    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/logback"/>
            <mvc:mapping path="/read-header"/>
            <mvc:mapping path="/form/*"/>
            <bean class="com.xtuer.interceptor.MyInterceptor"></bean>
        </mvc:interceptor>
    </mvc:interceptors>
```

每个 `<mvc:interceptor>` 可以包含多个 `<mvc:mapping path="expression"/>`，expression 可以使用正则表达式。

访问 <http://localhost/logback>, <http://localhost/read-header>, <http://localhost/form/*> 则会被拦截器 `MyInterceptor` 拦截，在控制台输出 `my interceptor`，访问其它的 URL 不会被这个拦截器拦截。

## 3. 拦截器的使用案例

**为了防止表单重复提交，可以使用拦截器给表单生成 token:**

1. `Get` 访问表单 <http://localhost/user-form>，拦截器在 `postHandle()` 生成 token
2. 把 token 存储到 session 或者 redis
3. 在 model 里写入 token
4. 在模版里把从 model 得到 token 并写入 `<form>` 的隐藏域
5. `Post` 提交表单到 <http://localhost/user-form>
6. 拦截器在 `preHandle()` 从 request 读取 token 进行验证
   * token 存在，允许表单提交，并删除 token，返回 true
   * token 不存在，则不允许表单提交，返回 false









