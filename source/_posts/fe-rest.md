---
title: jQuery 的 REST 插件
date: 2016-10-10 16:58:14
tags: FE
---
使用 REST 风格提交请求时，Content-Type 标准的来说应该用 **application/json**，但是服务器端获取请求的参数时必须从 Request Body 中获取，而且有些框架对从 Request Body 中获取数据支持不好，需要我们自己实现，SpringMvc 中使用注解 **@RequestBody** 从 Request Body 中获取数据。

SpringMvc 还提供了一个 Filter **HiddenHttpMethodFilter**，把 Content-Type 为 **application/x-www-form-urlencoded** 的 POST 请求，参数中 **_method** 值为 **PUT** 的请求分发为 PUT 请求，为 **DELETE** 请求分发为 DELETE 请求，实现了普通表单的 REST 风格提交，这样就可以使用 **@RequestParam** 获取参数的值了。

下面结合 SpringMvc 来介绍:

* Content-Type 为 application/x-www-form-urlencoded + HiddenHttpMethodFilter
  * 优点: 服务器端 GET, PUT, POST, DELETE 时直接参数映射为对象，或则都使用 @RequestParam 获取参数，使用形式一致、简洁
  * 缺点: 
    * 不是标准的 REST 规范
    * 参数是按照 key/value 的形式发送的，和普通表单的参数形式一样，有兴趣的可以在 Chrome 的 Network 中查看请求的 Headers
    * 不能传递复杂对象，因为只是简单的 key/value，不过非特殊情况也够用了，普通表单能做的它都能做
    * PUT 时参数中需要带上 _method="PUT"，DELETE 时参数中需要带上 _method="DELETE"
* Content-Type 为 application/json
  * 优点: 标准的 REST 规范，POST, PUT, DELETE 的参数是序列化后的 JSON 字符串，能够传递复杂的对象
  * 缺点: 
    * 服务器端直接参数映射为对象，或则 GET 时使用 @RequestParam 获取参数，POST, PUT, DELETE 使用 @RequestBody 获取参数到 Map 中，然后再从 Map 中获取一个一个的参数，非常繁琐
    * GET 和 POST, PUT, DELETE 获取参数的形式不统一，一个用 @RequestParam，其他的用 @RequestBody，需要脑子转换一下
    * 还有就是浏览器端 PUT, POST, DELETE 传递的 JSON 对象需要序列化后才能传给服务器端，可以使用 JSON.stringify(jsonObject) 进行序列化

总结下来，在 SpringMvc 中推荐使用 application/x-www-form-urlencoded + HiddenHttpMethodFilter 的方式实现 REST 的请求。<!--more-->

下面针对两种情况实现 rest 的插件:

* 实现一: application/x-www-form-urlencoded + HiddenHttpMethodFilter `推荐使用`
* 实现二: application/json `作为参考`

## 实现一

### 测试页面:

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>REST</title>
</head>

<body>

    <script src="http://cdn.bootcss.com/jquery/3.2.1/jquery.min.js"></script>
    <script src="/js/jquery.rest.js"></script>
    <script>
        var url = '/rest';

        $.rest.get({url: url, data: {name: 'Alice'}, success: function(result) {
            console.log(result);
        }});
        $.rest.create({url: url, data: {name: '张飞'}, success: function(result) {
            console.log(result);
        }});
        $.rest.update({url: url, data: {name: '关羽'}, success: function(result) {
            console.log(result);
        }});
        $.rest.remove({url: url, data: {name: '刘备'}, success: function(result) {
            console.log(result);
        }});
    </script>
</body>

</html>
```

输出:

```
{code: 0, data: "刘备", message: "DELETE", success: true}
{code: 0, data: "张飞", message: "POST", success: true}
{code: 0, data: "Alice", message: "GET", success: true}
{code: 0, data: "关羽", message: "PUT", success: true}
```

### REST 插件 jquery.rest.js:

```js
'use strict';

(function($) {
    /**
     * 执行 REST 请求的 jQuery 插件，sync 开头的为同步请求，不以 sync 开头的为异步请求:
     *      Get    请求调用 $.rest.get(),    $.rest.syncGet()
     *      Create 请求调用 $.rest.create(), $.rest.syncCreate()
     *      Update 请求调用 $.rest.update(), $.rest.syncUpdate()
     *      Delete 请求调用 $.rest.remove(), $.rest.syncRemove()
     *
     * 调用例子:
     *      $.rest.get({url: '/rest', data: {name: 'Alice'}, success: function(result) {
     *          console.log(result);
     *      }});
     *
     *      $.rest.syncGet({url: '/rest', data: {name: 'Alice'}, success: function(result) { // 同步请求
     *          console.log(result);
     *      }});
     *
     *      $.rest.update({url: '/rest/books/{bookId}', urlParams: {bookId: 23}, data: {name: 'C&S'}, success: function(result) {
     *          console.log(result);
     *      }}, fail: function(failResponse) {});
     */
    $.rest = {
        /**
         * 使用 Ajax 的方式执行 REST 的 GET 请求(服务器响应的数据根据 REST 的规范，必须是 Json 对象，否则转换为 JSON 失败导致进入错误的回调函数 fail).
         * 以下几个 REST 的函数 $.rest.create(), $.rest.update(), $.rest.remove() 只是请求的 HTTP 方法和 data 处理不一样，其他的都是相同的.
         *
         * @param {json} options 有以下几个选项:
         *        {string}   url       请求的 URL        (必选)
         *        {json}     urlParams URL 中的变量，例如 /rest/users/{id}，其中 {id} 为要被 urlParams.id 替换的部分 (可选)
         *        {json}     data      请求的参数         (可选)
         *        {boolean}  async     默认为异步方式     (可选)
         *        {function} success   请求成功时的回调函数(可选)
         *        {function} fail      请求失败时的回调函数(可选)
         *        {function} complete  请求完成后的回调函数(可选)
         * @return 没有返回值
         */
        get: function(options) {
            options.httpMethod = 'GET';
            this.sendRequest(options);
        },
        create: function(options) {
            options.data = options.data || {};
            options.httpMethod = 'POST';
            this.sendRequest(options);
        },
        update: function(options) {
            options.data = options.data || {};
            options.httpMethod   = 'POST';
            options.data._method = 'PUT'; // SpringMvc HiddenHttpMethodFilter 的 PUT 请求
            this.sendRequest(options);
        },
        remove: function(options) {
            options.data = options.data || {};
            options.httpMethod   = 'POST';
            options.data._method = 'DELETE'; // SpringMvc HiddenHttpMethodFilter 的 DELETE 请求
            this.sendRequest(options);
        },
        // 阻塞请求
        syncGet: function(options) {
            options.async = false;
            this.get(options);
        },
        syncCreate: function(options) {
            options.async = false;
            this.create(options);
        },
        syncUpdate: function(options) {
            options.async = false;
            this.update(options);
        },
        syncRemove: function(options) {
            options.async = false;
            this.remove(options);
        },

        /**
         * 执行 Ajax 请求，不推荐直接调用这个方法.
         *
         * @param {json} options 有以下几个选项:
         *        {string}   url        请求的 URL        (必选)
         *        {string}   httpMethod 请求的方式，有 GET, PUT, POST, DELETE (必选)
         *        {json}     urlParams  URL 中的变量      (可选)
         *        {json}     data       请求的参数        (可选)
         *        {boolean}  async      默认为异步方式     (可选)
         *        {function} success    请求成功时的回调函数(可选)
         *        {function} fail       请求失败时的回调函数(可选)
         *        {function} complete   请求完成后的回调函数(可选)
         */
        sendRequest: function(options) {
            var defaults = {
                data: {},
                async: true,
                success: function() {},
                fail: function(error) { console.error(error) }, // 默认把错误打印到控制台
                complete: function() {}
            };

            // 使用 jQuery.extend 合并参数
            var settings = $.extend({}, defaults, options);

            // 替换路径中的变量，例如 /rest/users/{id}, 其中 {id} 为要被 settings.urlParams.id 替换的部分
            if (settings.urlParams) {
                settings.url = settings.url.replace(/\{\{|\}\}|\{(\w+)\}/g, function(m, n) {
                    if (m == '{{') { return '{'; }
                    if (m == '}}') { return '}'; }
                    return settings.urlParams[n];
                });
            }

            // 执行 AJAX 请求
            $.ajax({
                url:   settings.url,
                data:  settings.data,
                async: settings.async,
                type:  settings.httpMethod,
                dataType:    'json',
                contentType: 'application/x-www-form-urlencoded',
                // 服务器抛异常时，有时 Windows 的 Tomcat 环境下竟然取不到 header X-Requested-With, Mac 下没问题，
                // 正常请求时是好的，手动添加 X-Requested-With 后正常和异常时都能取到了
                headers: {'X-Requested-With': 'XMLHttpRequest'}
            })
            .done(function(data, textStatus, jqXHR) {
                settings.success(data, textStatus, jqXHR);
            })
            .fail(function(jqXHR, textStatus, failThrown) {
                // data|jqXHR, textStatus, jqXHR|failThrown
                settings.fail(jqXHR, textStatus, failThrown);
            })
            .always(function() {
                settings.complete();
            });
        }
    };
})(jQuery);
```

### 服务器端:

添加下面的 Filter 到 web.xml, **servlet-name** 为 DispatcherServlet 的 servlet-name，根据自己的配置进行修改:

```xml
<!-- 浏览器的 form 不支持 put, delete 等 method, 由该 filter 将 /blog?_method=delete 转换为标准的 http delete 方法 -->
<filter>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <servlet-name>springmvc</servlet-name>
</filter-mapping>
```

Controller 的实现:

```java
package com.xtuer.controller;

import com.xtuer.bean.Result;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

@Controller
public class RestController {
    @GetMapping("/rest")
    @ResponseBody
    public Result get(@RequestParam String name) {
        return Result.ok("GET", name);
    }

    @PostMapping("/rest")
    @ResponseBody
    public Result post(@RequestParam String name) {
        return Result.ok("POST", name);
    }

    @PutMapping("/rest")
    @ResponseBody
    public Result put(@RequestParam String name) {
        return Result.ok("PUT", name);
    }

    @DeleteMapping("/rest")
    @ResponseBody
    public Result delete(@RequestParam String name) {
        return Result.ok("DELETE", name);
    }
}
```

## 实现二

使用 REST 风格调用 `jQuery.ajax()` 更新用户名:

```js
$.ajax({
    url: '/users/1/username',
    data: JSON.stringify({name: 'Bob'}),
    type: 'PUT',
    dataType: 'json',
    contentType: 'application/json'
})
.done(function(result) {
    console.log(result);
});
```

如果每个 REST 的请求都像上面这样写一遍: `PUT`, `POST`, `DELETE` 时需要 `JSON.stringify(data)`, 请求不同时 type 也不同，dataType 和 contentType 是固定的，这么多限制，很容易出错。

为了方便使用和减少错误的发生，我们可以把它简单的封装为一个 jQuery 的插件，使用更有语义的函数进行 RESTful 风格的访问，例如:

```js
$.rest.get({url: '/rest', data: {name: 'Alice'}, success: function(result) {
    console.log(result);
}});
```

### 测试页面:

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>REST</title>
</head>

<body>

    <script src="http://cdn.bootcss.com/jquery/3.2.1/jquery.min.js"></script>
    <script src="/js/jquery.rest.js"></script>
    <script>
        var url = '/rest';

        $.rest.get({url: url, data: {name: 'Alice'}, success: function(result) {
            console.log(result);
        }});
        $.rest.create({url: url, data: {name: '张飞'}, success: function(result) {
            console.log(result);
        }});
        $.rest.update({url: url, data: {name: '关羽'}, success: function(result) {
            console.log(result);
        }});
        $.rest.remove({url: url, data: {name: '刘备'}, success: function(result) {
            console.log(result);
        }});
    </script>
</body>

</html>

```

输出:

```
{code: 0, data: "刘备", message: "DELETE", success: true}
{code: 0, data: "张飞", message: "POST", success: true}
{code: 0, data: "Alice", message: "GET", success: true}
{code: 0, data: "关羽", message: "PUT", success: true}
```

> 前端的代码和实现一的完全一样。

### REST 插件 jquery.rest.js:

```js
'use strict';

(function($) {
    /**
     * 执行 REST 请求的 jQuery 插件:
     *      Get    请求调用 $.rest.get()
     *      Create 请求调用 $.rest.create()
     *      Update 请求调用 $.rest.update()
     *      Delete 请求调用 $.rest.remove()
     *
     * 调用例子:
     *      $.rest.get({url: '/rest', data: {name: 'Alice'}, success: function(result) {
     *          console.log(result);
     *      }}, fail: function(failResponse) {});
     */
    $.rest = {
        /**
         * 使用 Ajax 的方式执行 REST 的 GET 请求(服务器响应的数据根据 REST 的规范，必须是 Json 对象).
         * 以下几个 REST 的函数 $.rest.create(), $.rest.update(), $.rest.remove() 只是请求的 HTTP 方法和 data 处理不一样，其他的都是相同的.
         *
         * @param {json} options 有以下几个选项:
         *        {string}   url       请求的 URL        (必选)
         *        {json}     urlParams URL 中的变量，例如 /rest/users/{id}，其中 {id} 为要被 urlParams.id 替换的部分 (可选)
         *        {json}     data      请求的参数         (可选)
         *        {boolean}  async     默认为异步方式     (可选)
         *        {function} success   请求成功时的回调函数(可选)
         *        {function} fail      请求失败时的回调函数(可选)
         *        {function} complete  请求完成后的回调函数(可选)
         * @return 没有返回值
         */
        get: function(options) {
            options.httpMethod = 'GET';
            this.sendRequest(options);
        },
        create: function(options) {
            options.httpMethod = 'POST';
            options.data = options.data ? JSON.stringify(options.data) : {};
            this.sendRequest(options);
        },
        update: function(options) {
            options.httpMethod = 'PUT';
            options.data = options.data ? JSON.stringify(options.data) : {};
            this.sendRequest(options);
        },
        remove: function(options) {
            options.httpMethod = 'DELETE';
            options.data = options.data ? JSON.stringify(options.data) : {};
            this.sendRequest(options);
        },

        /**
         * 执行 Ajax 请求，不推荐直接调用这个方法.
         * 因为发送给 SpringMVC 的 Ajax 请求中必须满足下面的条件才能执行成功:
         *     1. dataType: 'json'
         *     2. contentType: 'application/json'
         *     3. data: GET 时为 json 对象，POST, PUT, DELETE 时必须为 JSON.stringify(data)
         *
         * @param {json} options 有以下几个选项:
         *        {string}   url        请求的 URL        (必选)
         *        {string}   httpMethod 请求的方式，有 GET, PUT, POST, DELETE (必选)
         *        {json}     urlParams  URL 中的变量      (可选)
         *        {json}     data       请求的参数        (可选)
         *        {boolean}  async      默认为异步方式     (可选)
         *        {function} success    请求成功时的回调函数(可选)
         *        {function} fail       请求失败时的回调函数(可选)
         *        {function} complete   请求完成后的回调函数(可选)
         */
        sendRequest: function(options) {
            var defaults = {
                data: {},
                async: true,
                success: function() {},
                fail: function(error) { console.error(error) }, // 默认把错误打印到控制台
                complete: function() {}
            };

            // 使用 jQuery.extend 合并参数
            var settings = $.extend({}, defaults, options);

            // 替换路径中的变量，例如 /rest/users/{id}, 其中 {id} 为要被 settings.urlParams.id 替换的部分
            if (settings.urlParams) {
                settings.url = settings.url.replace(/\{\{|\}\}|\{(\w+)\}/g, function(m, n) {
                    if (m == '{{') { return '{'; }
                    if (m == '}}') { return '}'; }
                    return settings.urlParams[n];
                });
            }

            // 执行 AJAX 请求
            $.ajax({
                url:   settings.url,
                data:  settings.data,
                async: settings.async,
                type:  settings.httpMethod,
                dataType:    'json',
                contentType: 'application/json;charset=utf-8',
                // 服务器抛异常时，有时 Windows 的 Tomcat 环境下竟然取不到 header X-Requested-With, Mac 下没问题，
                // 正常请求时是好的，手动添加 X-Requested-With 后正常和异常时都能取到了
                headers: {'X-Requested-With': 'XMLHttpRequest'}
            })
            .done(function(data, textStatus, jqXHR) {
                settings.success(data, textStatus, jqXHR);
            })
            .fail(function(jqXHR, textStatus, failThrown) {
                // data|jqXHR, textStatus, jqXHR|failThrown
                settings.fail(jqXHR, textStatus, failThrown);
            })
            .always(function() {
                settings.complete();
            });
        }
    };
})(jQuery);
```

### 服务器端:

```java
package com.xtuer.controller;

import com.xtuer.bean.Result;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

import java.util.Map;

@Controller
public class RestController {
    @GetMapping("/rest")
    @ResponseBody
    public Result get(@RequestParam String name) {
        return Result.ok("GET", name);
    }

    @PostMapping("/rest")
    @ResponseBody
    public Result post(@RequestBody Map<String, String> map) {
        String name = map.get("name");
        return Result.ok("POST", name);
    }

    @PutMapping("/rest")
    @ResponseBody
    public Result put(@RequestBody Map<String, String> map) {
        String name = map.get("name");
        return Result.ok("PUT", name);
    }

    @DeleteMapping("/rest")
    @ResponseBody
    public Result delete(@RequestBody Map<String, String> map) {
        String name = map.get("name");
        return Result.ok("DELETE", name);
    }
}
```

> 这种实现是不是发现服务器端获取参数很不方便？例如要求参数中必须有 name，这里还需要自己判断，如果使用 @RequestParam 的话则 SpringMvc 会给我们判断。

## Result.java

Result 用于统一服务器端返回的 JSON 格式。

```java
package com.xtuer.bean;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public final class Result<T> {
    private int code;        // 状态码，一般是当 success 为 true 或者 false 时不足够表达时可使用
    private boolean success; // 成功时为 true，失败时为 false
    private String message;  // 成功或则失败时的描述信息
    private T data;          // 成功或则失败时的更多详细数据，一般失败时不需要

    public Result(boolean success, String message) {
        this(success, message, null);
    }

    public Result(boolean success, String message, T data) {
        this.success = success;
        this.message = message;
        this.data = data;
    }

    public static Result ok() {
        return new Result(true, "success");
    }

    public static Result ok(String message) {
        return new Result(true, message);
    }

    public static <T> Result<T> ok(String message, T data) {
        return new Result(true, message, data);
    }

    public static Result fail(String message) {
        return new Result(false, message);
    }

    public static <T> Result<T> fail(String message, T data) {
        return new Result(false, message, data);
    }
}
```

## 思考

如果有必要，可以把这两种实现合并为一个，不同需要时通过指定 Content-Type 来调用对应的实现，这里就不再赘述了。

## 参考资料

为了理解 Content-Type 为 application/x-www-form-urlencoded 和 application/json 的区别，可以参考四种常见的 POST 提交数据方式 <https://imququ.com/post/four-ways-to-post-data-in-http.html>。