---
title: 异常处理
date: 2017-03-18 18:50:08
tags: SpringWeb
---

SpringMVC 提供了 3 种异常处理方法：

1. 使用 **@ExceptionHandler** 注解实现异常处理
2. 简单异常处理器 **SimpleMappingExceptionResolver**
3. 实现异常处理接口 **HandlerExceptionResolver**，自定义异常处理器

通过比较，实现异常处理接口 **HandlerExceptionResolver** 是最合适的，可以给定更多的异常信息，可一自定义异常显示页面等，下面介绍使用这种方式处理异常。<!--more-->

## 1. 定义自己的异常

实现自定义异常是为了能更灵活的添加一些默认异常没有的功能，例如制定异常显示的页面，如果需要更多功能，自行扩展。

```java
package com.xtuer.exception;

/**
 * 定义应用程序级别统一的异常，能够指定异常显示的页面，即 error view name。
 */
public class ApplicationException extends RuntimeException {
    private String errorViewName = null;

    public ApplicationException(String message) {
        this(message, null);
    }

    public ApplicationException(String message, String errorViewName) {
        super(message);
        this.errorViewName = errorViewName;
    }

    public String getErrorViewName() {
        return errorViewName;
    }
}
```

## 2. 异常处理类

异常没有被我们捕获，被抛给 Spring 后，Spring 会调用此异常处理类把异常默认显示在 error.htm，errorViewName 可以在上面的 ApplicationException 里定义，如果是 AJAX 访问，异常信息则以 AJAX 的方式返回给客户端，而不是显示在错误页面。

```java
package com.xtuer.exception;

import com.alibaba.fastjson.JSON;
import com.xtuer.bean.Result;
import com.xtuer.controller.UriView;
import com.xtuer.util.NetUtils;
import org.apache.commons.lang3.exception.ExceptionUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.ui.ModelMap;
import org.springframework.web.servlet.HandlerExceptionResolver;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * SpringMvc 使用的异常处理类，统一处理未捕捉的异常:
 * 1. 当 AJAX 请求时发生异常，返回 JSON 格式的错误信息
 * 2. 非 AJAX 请求时发生异常，错误信息显示到 HTML 网页
 */
public final class XHandlerExceptionResolver implements HandlerExceptionResolver {
    private static Logger logger = LoggerFactory.getLogger(XHandlerExceptionResolver.class);

    @Override
    public ModelAndView resolveException(HttpServletRequest request,
                                         HttpServletResponse response,
                                         Object handler, Exception ex) {
        String error = ex.getMessage();
        String stack = ExceptionUtils.getStackTrace(ex);

        // 异常记录到日志里，对于运维非常重要
        logger.warn(error);
        logger.warn(stack);

        return NetUtils.useAjax(request) ? handleAjaxException(response, error, stack)
                                         : handleNonAjaxException(ex, error, stack);
    }

    /**
     * 处理 AJAX 请求时的异常: 把异常信息使用 Result 格式化为 JSON 格式，以 AJAX 的方式写入到响应数据中。
     *
     * @param response HttpServletResponse 对象
     * @param error 异常的描述信息
     * @param stack 异常的堆栈信息
     * @return 返回 null，这时 SpringMvc 不会去查找 view，会根据 response 中的信息进行响应。
     */
    private ModelAndView handleAjaxException(HttpServletResponse response, String error, String stack) {
        Result<String> result = Result.fail(error, stack);
        NetUtils.ajaxResponse(response, JSON.toJSONString(result));
        return null;
    }

    /**
     * 处理非 AJAX 请求时的异常:
     * 1. 如果异常是 ApplicationException 类型的
     *    A. 如果指定了 errorViewName，则在 errorViewName 对应的网页上显示异常
     *    B. 如果没有指定 errorViewName，则在显示异常的默认页面显示异常
     * 2. 非 ApplicationException 的异常，即其它所有类型的异常，则在显示异常的默认页面显示异常
     *
     * @param ex 异常对象
     * @param error 异常的描述信息
     * @param stack 异常的堆栈信息
     * @return ModelAndView 对象，给定了 view 和异常信息
     */
    private ModelAndView handleNonAjaxException(Exception ex, String error, String stack) {
        String errorViewName = "error.htm"; // 显示错误的默认页面

        // 如果是我们定义的异常 ApplicationException，则取得它的异常显示页面的 view name
        if (ex instanceof ApplicationException) {
            ApplicationException appEx = (ApplicationException) ex;
            errorViewName = (appEx.getErrorViewName() == null) ? errorViewName : appEx.getErrorViewName();
        }

        ModelMap model = new ModelMap();
        model.addAttribute("error", error);  // 异常信息
        model.addAttribute("detail", stack); // 异常堆栈

        return new ModelAndView(errorViewName, model);
    }
}
```

```java
package com.xtuer.util;

import org.apache.commons.lang3.exception.ExceptionUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

public class NetUtils {
    private static Logger logger = LoggerFactory.getLogger(NetUtils.class);

    /**
     * 判断请求是否 AJAX 请求
     *
     * @param request HttpServletRequest 对象
     * @return 如果是 AJAX 请求则返回 true，否则返回 false
     */
    public static boolean useAjax(HttpServletRequest request) {
        return "XMLHttpRequest".equalsIgnoreCase(request.getHeader("X-Requested-With"));
    }

    /**
     * 使用 AJAX 的方式把响应写入 response 中
     *
     * @param response HttpServletResponse 对象，用于写入请求的响应
     * @param data 响应的数据
     */
    public static void ajaxResponse(HttpServletResponse response, String data) {
        response.setContentType("application/json"); // 使用 ajax 的方式
        response.setCharacterEncoding("UTF-8");

        try {
            // 写入数据到流里，刷新并关闭流
            PrintWriter writer = response.getWriter();
            writer.write(data);
            writer.flush();
            writer.close();
        } catch (IOException ex) {
            logger.warn(ExceptionUtils.getStackTrace(ex));
        }
    }
}
```

## 3. 在 spring-mvc.xml 里注册异常处理器

```xml
<!--<bean class="com.xtuer.exception.XHandlerExceptionResolver"/>-->
<bean id="compositeExceptionResolver" class="org.springframework.web.servlet.handler.HandlerExceptionResolverComposite">
    <property name="exceptionResolvers">
        <list>
            <bean class="com.xtuer.exception.XHandlerExceptionResolver"/>
        </list>
    </property>
    <property name="order" value="0"/>
</bean>
```

> `<bean class="com.xtuer.exception.XHandlerExceptionResolver"/>` 注册的异常处理器只能处理进入 Controller 中的方法体里抛出的异常，例如缺少参数，参数类型转换错误时发生异常则不会进入 Controller 的方法，这时抛出的异常则不能被这种方式注册的异常处理器处理。
>
> 如果需要所有的异常都能被我们的异常处理器处理，则需要按上面的方法使用 HandlerExceptionResolverComposite 来注册我们的异常处理器，并且 order 的值要足够小，order 越小优先级越高，因为 SpringMVC 有一个异常处理器链。

## 4. 测试用的 Controller

```java
@Controller
public class DemoController {
    // http://localhost:8080/exception
    @GetMapping("/exception")
    public String exception() {
        throw new RuntimeException("普通访问发生异常");
    }

    // http://localhost:8080/exception-ajax
    @GetMapping("/exception-ajax")
    @ResponseBody
    public Result exceptionWhenAjax(Demo demo) {
        System.out.println(JSON.toJSONString(demo));
        throw new RuntimeException("AJAX 访问发生异常");
    }
}
```

## 5. 显示错误的页面 error.htm

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title>错误</title>
    </head>
    <body>
        错误: ${error!}<br>

        <#if detail??>
            <pre>错误细节:<br>${detail}</pre>
        </#if>
    </body>
</html>
```

## 6. 测试

* <http://localhost:8080/exception>
* <http://localhost:8080/exception-ajax>