---
title: Spring 异步调用
date: 2016-04-29 16:37:29
tags: Spring
---

Spring 中让一个方法在新线程中运行，只需要给方法加上 `@Async` 注解就可以（当然也可以自己直接创建一个线程实现异步执行）。

典型的使用如用户注册后需要发送一封邮件进行验证，邮件发送完成的时间取决于很多因素，例如网络快的时候发的快一些，慢的时候需要多一些的时间。如果使用同步的方式发送邮件，有可能需要等很久邮件才发送完成然后用户才能得到注册成功的响应，体验不是很好，如果使用异步的方式发送邮件，在发邮件的同时用户就被告知注册成功，请去收件箱中查看邮件进行注册验证（异步的方式还可以使用 MQ）。

<!--more-->

## Spring 配置文件中启用异步
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:task="http://www.springframework.org/schema/task"
       xsi:schemaLocation="
       http://www.springframework.org/schema/task
       http://www.springframework.org/schema/task/spring-task.xsd">
    ...
    <task:annotation-driven/>
    ...
</beans>
```

## 需要异步执行的方法加上 `@Sync`
> 注意: 如果把 `@Async` 的方法定义在 Controller 中则异步不会生效

```java
package com.xtuer.service;

import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

@Service
public class AsyncService {
    /**
     * 需要在新线程中运行的方法，模拟耗时任务
     */
    @Async
    public void asyncMethod() {
        try {
            for (int i = 0; i < 10; ++i) {
                System.out.println("==> " + i);
                Thread.sleep(1000);
            }
        } catch (Exception ex) {}
    }
}
```

## Controller
```java
package com.xtuer.controller;

import com.xtuer.service.AsyncService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class AsyncController {
    @Autowired
    private AsyncService asyncService;

    @RequestMapping("/async")
    @ResponseBody
    public String async() {
        asyncService.asyncMethod();
        return "Async";
    }
}
```

## 测试
* 访问 <http://localhost:8080/async>
* 浏览器中马上看到输出 `Async`，请求完成了
* 控制台则在不停的输出，从 0 到 9

    ```
    ==> 0
    ==> 1
    ==> 2
    ==> 3
    ==> 4
    ==> 5
    ==> 6
    ==> 7
    ==> 8
    ==> 9
    ```

