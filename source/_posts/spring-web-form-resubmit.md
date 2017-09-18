---
title: 防止表单重复提交
date: 2017-03-31 18:26:26
tags: SpringWeb
---
开始介绍使用 redirect 技术防止表单提交，但是 redirect 解决不了后退到表单页面时重复提交表单，为了解决这个问题，加入了 token 的机制，而且同时能够防止 CRSF 攻击。

Token 的生成和校验也有不同的方式，如果每个 form 相关的处理方法中都写一遍 token 的生成和校验代码，在实际项目中是不太能接受的，文章的后面介绍了使用拦截器的方式生成和校验 token，这种方式在实际项目里才有意义。<!--more-->

## 1. 常规防止表单重复提交

1. `GET` 访问表单页面
2. 填写表单
3. `POST` 提交表单
4. Server 端处理表单数据，例如把数据写入数据库
5. 重定向到另一个页面，防止用户刷新页面重复提交表单

![](/img/spring-web/flash-attribute-1.png)

```html
<!-- 文件名: result.htm -->
Result: ${result!}
```

```html
<!-- 文件名: user-form.htm -->
<!DOCTYPE html>
<html>
<head>
    <title>Update User</title>
</head>
<body>
<form action="/user-form" method="post">
    Username: <input type="text" name="username"><br>
    Password: <input type="text" name="password"><br>
    <button type="submit">Update User</button>
</form>
</body>
</html>
```

```java
package com.xtuer.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

@Controller
public class ParameterController {
    // 显示表单
    @RequestMapping(value = "/user-form", method= RequestMethod.GET)
    public String showUserForm() {
        return "user-form.htm";
    }

    // 更新 User，把操作结果保存到 redirectAttributes，
    // redirect 到 result 页面显示操作结果
    @RequestMapping(value = "/user-form", method= RequestMethod.POST)
    public String handleUserForm(@RequestParam String username,
                                 @RequestParam String password,
                                 final RedirectAttributes redirectAttributes) {
        // Update user in database...
        System.out.println("Username: " + username + ", Password: " + password);

        // 操作结果显示给用户
        redirectAttributes.addFlashAttribute("result", "The user is already successfully updated");

        return "redirect:/result"; // URI instead of viewName
    }

    // 显示表单处理结果
    @RequestMapping("/result")
    public String result() {
        return "result.htm";
    }
}
```

------

**测试一:**

1. 访问 <http://localhost/user-form>  
   ![](/img/spring-web/Unresubmit-1.png)
2. 点击 `Update User`，表单成功提交后被重定向到 result 页面  
   ![](/img/spring-web/Unresubmit-2.png)
3. 刷新 result 页面，表单没有被重复提交，实现了防止表单重复提交的功能

## 2. 使用 token 进一步加强防止表单重复提交

但是，如果在浏览器里点击后退按钮后退到表单页面，点击 `Update User`，表单被再次提交了。可以使用 token 防止后退的情况下重复提交表单，访问表单页面的时候生成一个 token 在 form 里并且在 Server 端存储这个 token，提交表单的时候先检查 Server 端有没有这个 token，如果有则说明是第一次提交表单，然后把 token 从 Server 端删除，处理表单，redirect 到 result 页面，如果 Server 端没有这个 token，则说明是重复提交的表单，不处理表单的提交。

```html
<!-- 文件名: result.htm -->
Result: ${result!}
```

在 form 里增加一个 input 域存放 token(没有隐藏是为了方便测试):

```html
<!-- 文件名: user-form.htm -->
<!DOCTYPE html>
<html>
<head>
    <title>Update User</title>
</head>
<body>
<form action="/user-form" method="post">
    <input type="input" name="token" value="${token!}"><br>
    Username: <input type="text" name="username"><br>
    Password: <input type="text" name="password"><br>
    <button type="submit">Update User</button>
</form>
</body>
</html>
```

```java
package com.xtuer.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

import javax.servlet.http.HttpSession;
import java.util.UUID;

@Controller
public class ParameterController {
    // 显示表单
    @RequestMapping(value = "/user-form", method= RequestMethod.GET)
    public String showUserForm(ModelMap model, HttpSession session) {
        String token = UUID.randomUUID().toString().toUpperCase().replaceAll("-", "");

        model.addAttribute("token", token);
        session.setAttribute(token, token);

        return "user-form.htm";
    }

    // 更新 User，把操作结果保存到 redirectAttributes，
    // redirect 到 result 页面显示操作结果
    @RequestMapping(value = "/user-form", method= RequestMethod.POST)
    public String handleUserForm(@RequestParam String username,
                                 @RequestParam String password,
                                 @RequestParam String token,
                                 HttpSession session,
                                 RedirectAttributes redirectAttributes) {
        // 处理表单前，查看 token 是否有效
        if (token == null || token.isEmpty() || !token.equals(session.getAttribute(token))) {
            throw new RuntimeException("重复提交表单");
        }

        // 正常提交表单，删除 token
        session.removeAttribute(token);

        // Update user in database...
        System.out.println("Username: " + username + ", Password: " + password);

        // 操作结果显示给用户
        redirectAttributes.addFlashAttribute("result", "The user is already successfully updated");

        return "redirect:result";
    }

    // 显示表单处理结果
    @RequestMapping("/result")
    public String result() {
        return "result.htm";
    }
}
```

**测试二:**

1. 访问 <http://localhost/user-form>  
   ![](/img/spring-web/Unresubmit-3.png)
2. 点击 `Update User`，表单成功提交后被重定向到 result 页面  
   ![](/img/spring-web/Unresubmit-4.png)
3. 刷新 result 页面，表单没有被重复提交，实现了防止表单重复提交的功能
4. 点击后退按钮回到表单页面，点击 `Update User`，因为 token 不存在了，程序抛出异常，防止了表单的重复提交（显示异常页面不是最好的办法，更友好的做法是显示一个表单重复提交提示页面）  
   ![](/img/spring-web/Unresubmit-5.png)

## 3. 使用 SpringMVC 拦截器生成和验证 token

思考一下，为了给 user-form 增加 token，在处理 user-form 的方法里新加了很多代码，如果有 10 个 form, 100 个 form 都要使用 token 的机制呢？难道要去每个 form 处理的方法里都加上上面的那么多代码吗？上面 token 使用的是 UUID，如果要改成 static 类型的整数，每次生成时都加 1 呢？ token 存储在 session 里，项目进行到一定的时候要决定存储在第三方缓存如 Redis 里呢？每次需求的变更都要修改所有 form 的处理方法？ 工作量也太大了，谁遇到这样的问题都会抓狂，难怪招聘里着重强调：不许打项目经理！

**幸好 SpringMVC 提供了拦截器的机制，能够很简单的给 form 增加 token 的机制:**

* 当访问 user-form 页面时，在拦截器的 **postHandle()** 里生成 token 存放在 ModelAndView 和 session 里，然后把 token 写到 form 表单中
* 提交表单时，在拦截器的 **preHandle()** 里校验 token，如果 token 无效则禁止表单的提交
* 在拦截器的配置里加上需要使用 token 机制的 form 的路径
* 如果 form 不需要 token 机制，从拦截器的配置里把它的路径删除即可
* 不需要修改 Controller 中的代码

> 什么是 token? 简单的说就是一次操作的标识，可以是数字，字符串，甚至对象等，只要能把不同的表单提交区别开来就可以了。申请表单的时候生成一个 token，表单提交后删除 token。

```html
<!-- 文件名: result.htm -->
Result: ${result!}
```

在 form 里增加一个 input 域存放 token:

```html
<!-- 文件名: user-form.htm -->
<!DOCTYPE html>
<html>
<head>
    <title>Update User</title>
</head>
<body>
<form action="/user-form" method="post">
    <input type="input" name="token" value="${token!}"><br>
    Username: <input type="text" name="username"><br>
    Password: <input type="text" name="password"><br>
    <button type="submit">Update User</button>
</form>
</body>
</html>
```

和开始的 Controller 代码一样，没有 token 的相关代码:

```java
package com.xtuer.comcontroller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

@Controller
public class ParameterController {
    // 显示表单
    @RequestMapping(value = "/user-form", method= RequestMethod.GET)
    public String showUserForm() {
        return "user-form.htm";
    }

    // 更新 User，把操作结果保存到 redirectAttributes，
    // redirect 到 result 页面显示操作结果
    @RequestMapping(value = "/user-form", method= RequestMethod.POST)
    public String handleUserForm(@RequestParam String username,
                                 @RequestParam String password,
                                 final RedirectAttributes redirectAttributes) {
        // Update user in database...
        System.out.println("Username: " + username + ", Password: " + password);

        // 操作结果显示给用户
        redirectAttributes.addFlashAttribute("result", "The user is already successfully updated");

        return "redirect:result";
    }

    // 显示表单处理结果
    @RequestMapping("/result")
    public String result() {
        return "result.htm";
    }
}
```

拦截器 TokenValidator 用于生成和校验 token:

```java
package com.xtuer.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.UUID;

public class TokenValidator implements HandlerInterceptor {
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) throws Exception {
        // POST, PUT, DELETE 请求都有可能是表单提交
        if (!"GET".equalsIgnoreCase(request.getMethod())) {
            String clientToken = request.getParameter("token");
            String serverToken = (String) request.getSession().getAttribute(clientToken);

            if (clientToken == null || clientToken.isEmpty() || !clientToken.equals(serverToken)) {
                throw new RuntimeException("重复提交表单");
            }

            // 正常提交表单，删除 token
            request.getSession().removeAttribute(clientToken);
        }

        return true;
    }

    public void postHandle(HttpServletRequest request,
                           HttpServletResponse response,
                           Object handler,
                           ModelAndView modelAndView) throws Exception {
        // GET 请求访问表单页面
        if (!"GET".equalsIgnoreCase(request.getMethod())) {
            return;
        }

        // 生成 token 存储到 session 里，并且保存到 form 的 input 域
        String token = UUID.randomUUID().toString().replaceAll("-", "").toUpperCase();

        modelAndView.addObject("token", token);
        request.getSession().setAttribute(token, token);
    }

    public void afterCompletion(HttpServletRequest request,
                                HttpServletResponse response,
                                Object handler,
                                Exception ex) throws Exception {

    }
}
```

spring-mvc.xml 里配置拦截器

```xml
<beans>
    ...
    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/user-form"/> <!--需要增加 token 校验的 form 的 URI-->
            <bean class="com.xtuer.interceptor.TokenValidator"></bean>
        </mvc:interceptor>
    </mvc:interceptors>
</beans>
```

**测试三:**

1. 访问 <http://localhost/user-form>  
   ![](/img/spring-web/Unresubmit-3.png)
2. 点击 `Update User`，表单成功提交后被重定向到 result 页面  
   ![](/img/spring-web/Unresubmit-4.png)
3. 刷新 result 页面，表单没有被重复提交，实现了防止表单重复提交的功能
4. 点击后退按钮回到表单页面，点击 `Update User`，因为 token 不存在了，程序抛出异常，防止了表单的重复提交（显示异常页面不是最好的办法，更友好的做法是显示一个表单重复提交提示页面）  
   ![](/img/spring-web/Unresubmit-5.png)

**通过 SpringMVC 拦截器增加 token 的机制，**

* 想改变 token 生成策略？ 修改 TokenValidator
* 想改变 token 的存储策略？ 修改 TokenValidator
* 想给 form 增加 token 校验？ 修改 spring-mvc.xml 拦截器的配置
* 想把 form 的 token 校验删除？ 修改 spring-mvc.xml 拦截器的配置
* 不需要修改任何 form 处理的方法，泰山崩于前而色不变，风波骤起而泰然处之，项目经理好像也没那么可恨了

## 思考

Token 的存储需要考虑过期时间，否则访问 10 万次 user-form 页面，生成 10 万个 token 而不提交表单，token 一直不会被删除，会造成很大的资源浪费。

Token 应该写入到 form 的隐藏域，为了直观，上面我们写入了普通的 input 中：
`<input type="hidden" name="token" value="${token!}">`

这里使用的是 SpringMVC 的拦截器生成和校验 token，当然也可以使用 Servlet 的 Filter 等技术实现。

重复提交表单时不应该直接把异常显示给用户，可以使用 SpringMVC 的异常处理机制，不同的页面显示不同异常的友好信息。