---
title: SpringMVC 进一步学习
date: 2017-03-31 15:58:26
tags: SpringWeb
---

## @PathVariable

取得 `URL` 路径中的匹配的内容，适合 RESTful 的风格。

> A `@PathVariable` argument can be of any simple type such as int, long, Date, etc. Spring automatically converts to the appropriate type or throws a TypeMismatchException if it fails to do so. You can also register support for parsing additional data types.

```java
package com.xtuer.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class ParameterController {
    // {userId} 是 placeholder，里面的内容可以用 @PathVariable 取到
    @RequestMapping("/users/{userId}")
    @ResponseBody
    public String one(@PathVariable Integer userId) {
        return userId + "";
    }

    // 如果变量名和 {} 中的名字不一样，
    // 需要用 @PathVariable("nameInPath") 来指定变量要使用路径中的哪一个变量
    @RequestMapping("/categories/{categoryName}/products/{productId}")
    @ResponseBody
    public String two(@PathVariable String categoryName,
                      @PathVariable("productId") Integer pId) {
        return "categoryName: " + categoryName + ", productId: " + pId;
    }
    
    // 还支持正则表达式的方式
    @RequestMapping("/regex/{text:[a-z]+}-{number:\\d+}")
    @ResponseBody
    public String three(@PathVariable String text, @PathVariable Integer number) {
        return "Text: " + text + ", Number: " + number;
    }
}
```

**测试：**

* <http://localhost/users/1234>
* <http://localhost/categories/xbox/products/1234>
* <http://localhost/regex/part-1234> <!--more-->

## @RequestParam

取得 `HTTP 请求`中的参数。

```java
package com.xtuer.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class ParameterController {
    // 取得名字为 id 的参数的值
    @RequestMapping("/user")
    @ResponseBody
    public String findUser(@RequestParam Integer id) {
        return "ID: " + id;
    }

    // required 默认是 true，参数是必要的，如果没有提供需要的参数，则报错
    // required 为 false 表示参数是可选的
    @RequestMapping("/product")
    @ResponseBody
    public String findProduct(@RequestParam(value="productId", required=true) Integer id,
                              @RequestParam(value="productName", required=false) String name) {
        return "ID: " + id + ", Name: " + name;
    }
}
```

**测试：**

* <http://localhost/user?id=1234>
* <http://localhost/product?productId=1234&productName=PS4>

## @ModelAttribute

把 `HTTP 请求`的参数映射到对象，参数名和对象中的属性名匹配的就做映射，不匹配的就不管。

```java
package com.xtuer.controller;

import com.xtuer.domain.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class ParameterController {
    // 表单中的参数被映射到对象 user
    @RequestMapping("/user")
    @ResponseBody
    public String findUser(@ModelAttribute User user,
                           @RequestParam(required=false) Integer age) {
        System.out.println("Age: " + age);
        return user.toString();
    }
}
```

```java
package com.xtuer.domain;

import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

@Getter
@Setter
@ToString
public class User {
    private int id;
    private String username;
    private String password;
}
```

**测试：**

* <http://localhost/user?id=1234&username=biao>
* <http://localhost/user?id=1234&username=biao&password=Secret>
* <http://localhost/user?id=1234&username=biao&password=Secret&age=12>

## Forward and Redirect

```java
package com.xtuer.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class ParameterController {
    @RequestMapping("/forward-test")
    public String forward() {
        return "forward:/helloworld-springmvc";
    }

    @RequestMapping("/redirect-test")
    public String redirect() {
        return "redirect:/helloworld-springmvc";
    }
}
```

**测试：**

* <http://localhost/forward-test>
* <http://localhost/redirect-test>

## RedirectAttributes

表单提交后一般都会 redirect 到另一个页面，防止表单重复提交。**RedirectAttributes** 的作用就是把处理 PageA 的结果存储起来，当 redirect 到 PageB 的时候显示 PageA 的结果。

Request 中的参数不能被传递给 redirect 的页面，因为 redirect 是从浏览器端发起一个新的请求。

> Normally when we generate an http redirect request, the data stored in request is lost making it impossible for next GET request to access some of the useful information from request.
>
> Flash attributes comes handy in such cases. Flash attributes provide a way for one request to store attributes intended for use in another.
> Flash attributes are saved temporarily (typically in the session) before the redirect to be made available to the request after the redirect and removed immediately.

**防止表单重复提交的流程**:
![](/img/spring-web/flash-attribute-1.png)

```html
<!-- 文件名: view/user-form.htm -->
<!DOCTYPE html>
<html>
<head>
    <title>Update User</title>
</head>
<body>
    <form action="/update-user" method="post">
        Username: <input type="text" name="username"><br>
        Password: <input type="text" name="password"><br>
        <button type="submit">Update User</button>
    </form>
</body>
</html>
```

```html
<!-- 文件名: view/result.htm -->
Result: ${result}
```

```java
package com.xtuer.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

@Controller
public class ParameterController {
    // 显示表单
    @RequestMapping("/user-form")
    public String showUserForm() {
        return "user-form.htm";
    }

    // 更新 User，把结果保存到 RedirectAttributes
    @RequestMapping("/update-user")
    public String updateUser(@RequestParam String username,
                             @RequestParam String password,
                             final RedirectAttributes redirectAttributes) {
        // Update user in database...
        System.out.println("Username: " + username + ", Password: " + password);

        // 操作结果显示给用户
        redirectAttributes.addFlashAttribute("result", "The user is already successfully updated");

        return "redirect:/result";
    }

    // 显示表单处理结果
    @RequestMapping("/result")
    public String result() {
        return "result.htm";
    }
}
```

**测试：**

访问 <http://localhost/user-form>，填写信息，提交表单，处理好后页面被 redirect 到 <http://localhost/result> 显示操作结果。

![](/img/spring-web/flash-attribute-2.png)

------

Redirect 到另一个 Controller 时获取 RedirectAttributes 里的属性使用 **@ModelAttribute**

```java
    @RequestMapping("/flash")
    public String flash(RedirectAttributes redirectAttributes) {
        redirectAttributes.addFlashAttribute("username", "Biao");
        return "redirect:flash2";
    }

    @RequestMapping("/flash2")
    @ResponseBody
    public String flash2(@ModelAttribute("username") String username) {
        return "username: " + username;
    }
```

## 获取 Request and Response

想取得 HttpServletRequest 和 HttpServletResponse 很容易，只要在方法的参数里定义后，SpringMVC 会自动的注入 它们。

```java
package com.xtuer.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Controller
public class ParameterController {
    @RequestMapping("/request-response")
    @ResponseBody
    public String foo(HttpServletRequest request, HttpServletResponse response) {
        System.out.println(request);
        return "WoW";
    }
}
```

**测试：**

* <http://localhost/request-response>

## Header

* Read header: The `@RequestHeader` annotation allows a method parameter to be bound to a request header.
* Write header: Header 直接写入到 `HttpServletResponse`，没有注解用来写 header

| Header          | Value                                    |
| --------------- | ---------------------------------------- |
| Host            | localhost:8080                           |
| Accept          | text/html,application/xhtml+xml,application/xml;q=0.9 |
| Accept-Language | fr,en-gb;q=0.7,en;q=0.3                  |
| Accept-Encoding | gzip,deflate                             |
| Accept-Charset  | ISO-8859-1,utf-8;q=0.7,*;q=0.7           |
| Keep-Alive      | 300                                      |

```java
package com.xtuer.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestHeader;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletResponse;

@Controller
public class ParameterController {
    @RequestMapping("/read-header")
    @ResponseBody
    public String readHeader(@RequestHeader("Host") String host,
                             @RequestHeader(value="Accept-Encoding", required=false) String encoding,
                             @RequestHeader(value="Accept-Charset", required=false) String charset) {
        return "Host: " + host + ", Encoding: " + encoding + ", Charset: " + charset;
    }

    @RequestMapping("/read-header-error")
    @ResponseBody
    public String readHeaderError(@RequestHeader("Host") String host,
                                  @RequestHeader("Accept-Charset") String charset) {
        return "Host: " + host + ", Charset: " + charset;
    }

    @RequestMapping("/write-header")
    @ResponseBody
    public String writeHeader(HttpServletResponse response) {
        response.setHeader("token", "D4BFCEC2-89E6-40CB-AF9A-B5513CB30FED");

        return "Header is wrote.";
    }
}
```

**测试：**

* <http://localhost/read-header>
* <http://localhost/read-header-error>
* <http://localhost/write-header>

访问 <http://localhost/read-header> 成功访问  
![](/img/spring-web/request-header-1.png)

访问 <http://localhost/read-header-error> 提示错误，因为 charset 是 required=true的，在 header 信息里没有 charset，所以报错。但是这个错误如果不了解的话会一头雾水，因为在控制台中没有输出错误的原因  
![](/img/spring-web/request-header-1.png)

> 参数错误，类型转换错误等默认在控制台中看不到，也不会返回给浏览器端，要看到错误原因，需要把 SpringMv 的日志调到 Debug 级别。

## Cookie

* Read cookie: The `@CookieValue` annotation allows a method parameter to be bound to the value of an HTTP cookie.
* Write cookie: Cookie 直接写到 `HttpServletResponse`，没有注解用来写 cookie

```java
package com.xtuer.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.CookieValue;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServletResponse;

@Controller
public class ParameterController {
    @RequestMapping("/read-cookie")
    @ResponseBody
    public String readCookie(@CookieValue("username") String cookie) {
        return "Cookie for username: " + cookie;
    }

    @RequestMapping("/write-cookie")
    @ResponseBody
    public String writeCookie(HttpServletResponse response) {
        Cookie cookie = new Cookie("username", "Don't tell you");
        cookie.setMaxAge(1000);
        response.addCookie(cookie); // Put cookie in response.

        return "Cookie is wrote.";
    }
}
```

**测试：**

1. 访问 <http://localhost/read-cookie> 报错，因为还没有 cookie，
   可以给 cookie 一个默认值，这样即使没有 cookie 也不会报错了:
   `@CookieValue(value="username", defaultValue="") String cookie`

2. 访问 <http://localhost/write-cookie> 写入 cookie

3. 访问 <http://localhost/read-cookie> 输出 cookie

## Session

* 可以使用 `HttpSession` 来直接操作 session
* 同时也提供了操作 session 的注解 `@SessionAttributes`

HttpSession 的方式什么时候都生效，但是 @SessionAttributes 有时候就不行，如返回 AJAX 请求时设置的 session 无效。**所以推荐使用注入 HttpSession 来读写 session**。

> `@SessionAttributes` annotation indicates that in the controller’s methods can be assigned some values to arguments of the annotation. In this example I declared just one session attribute with the name “thought“. That’s mean I can put some object into modelAndView using addObject() method, and it will be added to the session if the name of the object will be the same as the name of argument in @SessionAttributes.

```java
package com.xtuer.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.SessionAttributes;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpSession;

@Controller
@SessionAttributes({"result"})
public class SessionController {
    @RequestMapping("/write-session")
    @ResponseBody
    public String writeSession(HttpSession session) {
        session.setAttribute("username", "座山雕");
        session.setAttribute("password", "天王盖地虎");

        return "Session wrote...";
    }

    @RequestMapping("/read-session")
    public String readSession() {
        return "helloworld-freemarker.htm";
    }

    @RequestMapping("/write-session2")
    public ModelAndView writeSession2() {
        ModelAndView mav = new ModelAndView();
        mav.addObject("result", "Save session");
        mav.setViewName("user-form.htm");

        return mav;
    }

    @RequestMapping("/read-session2")
    public String readSession2() {
        return "result.htm";
    }
}
```

**测试：**

1. 访问 <http://localhost/write-session>
2. 访问 <http://localhost/read-session>


## SpringMVC 的处理流程

1. 用户向服务器发送请求，请求被 Spring 前端控制 Servelt `DispatcherServlet` 捕获
2. DispatcherServlet 对请求 URL 进行解析，得到请求资源标识符（URI）。然后根据该 URI，调用`HandlerMapping` 获得该 Handler 配置的所有相关的对象（包括 Handler 对象以及 Handler 对象对应的拦截器），最后以 `HandlerExecutionChain` 对象的形式返回
3. DispatcherServlet 根据获得的 Handler，选择一个合适的 `HandlerAdapter`。（附注：如果成功获得HandlerAdapter 后，此时将开始执行拦截器的 preHandler(...)方法）
4. 提取 Request 中的模型数据，填充 Handler 入参，开始执行 Handler（Controller)。 在填充Handler 的入参过程中，根据你的配置，Spring 将帮你做一些额外的工作：
   `HttpMessageConveter`： 将请求消息（如 Json、xml 等数据）转换成一个对象，将对象转换为指定的响应信息 
   `数据转换`：对请求消息进行数据转换，如 String 转换成 Integer、Double 等  
   `格式化`：对请求消息进行数据格式化，如将字符串转换成格式化数字或格式化日期等  
   `数据验证`： 验证数据的有效性（长度、格式等），验证结果存储到 BindingResult 或 Error 中
5. Handler 执行完成后，向 DispatcherServlet 返回一个 `ModelAndView` 对象
6. 根据返回的 ModelAndView，选择一个适合的 `ViewResolver`（必须是已经注册到 Spring 容器中的ViewResolver）返回给 DispatcherServlet 
7. ViewResolver 结合 `Model` 和 `View`，来渲染视图
8. 将渲染结果返回给客户端

[Spring MVC 教程, 快速入门, 深入分析](http://elf8848.iteye.com/blog/875830/)

![](/img/spring-web/SpringMVC-Architecture-1.png)

![](/img/spring-web/SpringMVC-Architecture-2.png)

![](/img/spring-web/SpringMVC-Architecture-3.png)

![](/img/spring-web/SpringMVC-Architecture-4.png)

![](/img/spring-web/SpringMVC-Architecture-5.jpg)