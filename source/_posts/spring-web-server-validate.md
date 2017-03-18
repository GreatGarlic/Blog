---
title: 服务器端参数验证
date: 2017-03-18 16:09:28
tags: Spring-Web
---

服务器端接收到前端传递过来的参数后，很多情况下如果不做校验就直接使用是非常危险的，例如造成 XSS 攻击，把无效数据保存到数据库导致业务出错等。传统的参数校验方式一般是取得每一个参数，然后一个一个的使用 if else 和规则比较进行校验，这样就会有大量的、简单重复的校验代码散布在代码中，不利于维护和阅读。这里介绍使用注解，根据 **JSR-303 Validation** 进行参数验证。

当我们在 SpringMVC 中需要使用到 JSR-303 的时候就需要我们提供一个对 JSR-303 规范的实现，**Hibernate Validator** 实现了这一规范，下面将以它作为 JSR-303 的实现来讲解 SpringMVC 对 JSR-303 的支持。

**使用方法:**

* Bean 中使用 @NotNull 等定义验证规则
* Controller 中使用 @Valid 进行参数校验
* 如有参数错误，则返回错误信息给客户端

<!--more-->

## 校验规则

| Annocation                | Description                              |
| ------------------------- | ---------------------------------------- |
| @Null                     | 限制只能为null                                |
| @NotNull                  | 限制必须不为null                               |
| @AssertFalse              | 限制必须为false                               |
| @AssertTrue               | 限制必须为true                                |
| @DecimalMax(value)        | 限制必须为一个不大于指定值的数字                         |
| @DecimalMin(value)        | 限制必须为一个不小于指定值的数字                         |
| @Digits(integer,fraction) | 限制必须为一个小数，且整数部分的位数不能超过integer，小数部分的位数不能超过fraction |
| @Future                   | 限制必须是一个将来的日期                             |
| @Max(value)               | 限制必须为一个不大于指定值的数字                         |
| @Min(value)               | 限制必须为一个不小于指定值的数字                         |
| @Past                     | 限制必须是一个过去的日期                             |
| @Pattern(value)           | 限制必须符合指定的正则表达式                           |
| @Size(max,min)            | 限制字符长度必须在min到max之间                       |

## Gradle 依赖

```groovy
compile('org.hibernate:hibernate-validator:5.4.0.Final')
```

## Bean 中定义校验规则

类 Demo 的属性定义的时候给定校验规则，校验规则能够复合使用。

```java
package com.xtuer.bean;

import lombok.Getter;
import lombok.Setter;
import org.hibernate.validator.constraints.NotBlank;
import javax.validation.constraints.Min;
import javax.validation.constraints.NotNull;

@Getter
@Setter
public class Demo {
    @NotNull(message="ID 不能为 null")
    @Min(value=1, message="ID 不能小于 1")
    private Long id;

    @NotBlank(message="Info 不能为空")
    private String info;

    public Demo() {
    }

    public Demo(Long id, String info) {
        this.id = id;
        this.info = info;
    }
}
```

## Controller 中校验参数

**@Valid** 表示需要对其注解的对象进行参数校验。

```java
package com.xtuer.controller;

@Controller
public class DemoController {
    // http://localhost:8080/demo/validate
    // http://localhost:8080/demo/validate?id=2
    // http://localhost:8080/demo/validate?id=2&info=amazing
    @GetMapping(UriView.URI_DEMO_VALIDATE)
    @ResponseBody
    public Result validateDemo(@Valid Demo demo, BindingResult bindingResult) {
        // 如有参数错误，则返回错误信息给客户端
        if (bindingResult.hasErrors()) {
            return Result.fail(CommonUtils.getBindingMessage(bindingResult));
        }

        return Result.ok("", demo);
    }
}
```

```java
package com.xtuer.util;

import org.springframework.validation.BindingResult;
import org.springframework.validation.FieldError;

public class CommonUtils {
    /**
     * BindingResult 中的错误信息很多，对用户不够友好，使用 getBindingMessage()
     * 提取对用户阅读友好的定义验证规则 message。
     *
     * @param result 验证的结果对象
     * @return 验证规则 message
     */
    public static String getBindingMessage(BindingResult result) {
        StringBuffer sb = new StringBuffer();

        for (FieldError error : result.getFieldErrors()) {
            // sb.append(error.getField() + " : " + error.getDefaultMessage() + "\n");
            sb.append(error.getDefaultMessage() + "\n");
        }

        return sb.toString();
    }
}
```

## 参考资料

* [SpringMVC 介绍之 Validation](http://haohaoxuexi.iteye.com/blog/1812584)

