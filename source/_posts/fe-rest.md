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
  * 优点: 标准的 REST 规范，GET 处理和上面的一样，但是 POST, PUT, DELETE 的参数是序列化后的 JSON 字符串，能够传递复杂的对象
  * 缺点: 
    * 服务器端直接参数映射为对象，或则 GET 时使用 @RequestParam 获取参数，POST, PUT, DELETE 使用 @RequestBody 获取参数到 Map 中，然后再从 Map 中获取一个一个的参数，非常繁琐
    * GET 和 POST, PUT, DELETE 获取参数的形式不统一，一个用 @RequestParam，其他的用 @RequestBody，需要脑子转换一下
    * 还有就是浏览器端 PUT, POST, DELETE 传递的 JSON 对象需要序列化后才能传给服务器端，可以使用 JSON.stringify(jsonObject) 进行序列化

总结下来，在 SpringMvc 中推荐使用 application/x-www-form-urlencoded + HiddenHttpMethodFilter 的方式实现 REST 的请求，就是为了获取参数时比较统一，但是当需要传递复杂的参数时，例如属性是多层嵌套的对象，Json 对象的数组，这时再使用 application/json 的方式。

为了简化 Rest Ajax 的访问，下面对 jQuery 的 Ajax 进行了简单的封装，例如更新用户名原始实现大致如下:

```js
$.ajax({
    url        : '/users/1/username',
    data       : JSON.stringify({name: 'Bob'}),
    type       : 'PUT',
    dataType   : 'json',
    contentType: 'application/json'
})
.done(function(result) {
    console.log(result);
});
```

如果每个 REST 的请求都像上面这样写一遍: `PUT`, `POST`, `DELETE` 时需要 `JSON.stringify(data)`, 请求不同时 type 也不同，dataType 和 contentType 是固定的，这么多限制，很容易出错。使用下面实现的 rest 插件后，简化如下，只需要关心参数和回调，不需要处理其他额外信息，而且 $.rest.update 名字也更有语义化，一看就知道是更新操作:

```js
$.rest.update({
    url    : '/users/1/username', 
    data   : {name: 'Bob'}, 
    success: function(result) {
        console.log(result);
    }
});
```

<!--more-->

## 测试页面:

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
        // [1] 服务器端的 GET 需要启用 UTF-8 才不会乱吗
        $.rest.get({url: '/rest', data: {name: 'Alice'}, success: function(result) {
            console.log(result);
        }});

        // [2] 普通 form 表单提交 rest Ajax 请求
        $.rest.create({url: '/rest', success: function(result) {
            console.log(result);
        }});

        $.rest.update({url: '/rest', data: {name: '黄飞鸿', age: 22}, success: function(result) {
            console.log(result);
        }});

        $.rest.remove({url: '/rest', success: function(result) {
            console.log(result);
        }});

        // [3] 使用 request body 传递复杂 Json 对象
        $.rest.create({url: '/rest/requestBody', data: {name: 'Alice'}, jsonRequestBody: true, success: function(result) {
            console.log(result);
        }});

        $.rest.update({url: '/rest/requestBody', data: {name: 'Alice'}, jsonRequestBody: true, success: function(result) {
            console.log(result);
        }});

        $.rest.remove({url: '/rest/requestBody', data: {name: 'Alice'}, jsonRequestBody: true, success: function(result) {
            console.log(result);
        }});
    </script>
</body>

</html>
```

输出:

```
{code: 0, data: "Alice", message: "GET handled", success: true}

{code: 0, message: "CREATE handled", success: true}
{code: 0, message: "DELETE handled", success: true}
{code: 0, data: "黄飞鸿 : 22", message: "UPDATE handled", success: true}

{code: 0, message: "UPDATE requestBody handled: {\"name\":\"Alice\"}", success: true}
{code: 0, message: "CREATE requestBody handled: {\"name\":\"Alice\"}", success: true}
{code: 0, message: "DELETE requestBody handled: {\"name\":\"Alice\"}", success: true}
```

## REST 插件 jquery.rest.js:

```js
'use strict';

(function($) {
    /**
     * 执行 REST 请求的 jQuery 插件，不以 sync 开头的为异步请求，以 sync 开头的为同步请求:
     *      Get    请求调用 $.rest.get(),    $.rest.syncGet()
     *      Create 请求调用 $.rest.create(), $.rest.syncCreate()
     *      Update 请求调用 $.rest.update(), $.rest.syncUpdate()
     *      Delete 请求调用 $.rest.remove(), $.rest.syncRemove()
     *
     * 默认使用 contentType 为 application/x-www-form-urlencoded 的方式提交请求，只能传递简单的 key/value，
     * 就是普通的 form 表单提交，如果想要向服务器传递复杂的 json 对象，可以使用 contentType 为 application/json 的格式，
     * 只要设置请求的参数 jsonRequestBody 为 true 即可，例如
     *      $.rest.update({url: '/rest', data: {name: 'Alice'}, jsonRequestBody: true, success: function(result) {
     *          console.log(result);
     *      }});
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
         * 使用 Ajax 的方式执行 REST 的 GET 请求(服务器响应的数据根据 REST 的规范，必须是 Json 对象，否则浏览器端会解析出错).
         * 以下几个 REST 的函数 $.rest.create(), $.rest.update(), $.rest.remove() 只是请求的 HTTP 方法和 data 处理不一样，
         * 其他的都是相同的，所以就不再重复注释了。
         *
         * @param {Json} options 有以下几个选项:
         *               {String}   url       请求的 URL        (必选)
         *               {Json}     urlParams URL 中的变量，例如 /rest/users/{id}，其中 {id} 为要被 urlParams.id 替换的部分 (可选)
         *               {Json}     data      请求的参数         (可选)
         *               {Boolean}  jsonRequestBody 是否使用 application/json 的方式进行请求，默认为 false (可选)
         *               {Function} success   请求成功时的回调函数(可选)
         *               {Function} fail      请求失败时的回调函数(可选)
         *               {Function} complete  请求完成后的回调函数(可选)
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
         * @param {Json} options 有以下几个选项:
         *               {String}   url        请求的 URL        (必选)
         *               {String}   httpMethod 请求的方式，有 GET, PUT, POST, DELETE (必选)
         *               {Json}     urlParams  URL 中的变量      (可选)
         *               {Json}     data       请求的参数        (可选)
         *               {Boolean}  async      默认为异步方式     (可选)
         *               {Boolean}  jsonRequestBody 是否使用 application/json 的方式进行请求，默认为 false (可选)
         *               {Function} success    请求成功时的回调函数(可选)
         *               {Function} fail       请求失败时的回调函数(可选)
         *               {Function} complete   请求完成后的回调函数(可选)
         */
        sendRequest: function(options) {
            // 默认设置
            var defaults = {
                data: {},
                async: true,
                jsonRequestBody: false,
                contentType: 'application/x-www-form-urlencoded;charset=UTF-8',
                success: function() {},
                fail: function(error) { console.error(error) }, // 默认把错误打印到控制台
                complete: function() {}
            };

            // 使用 jQuery.extend 合并用户传递的 options 和 defaults
            var settings = $.extend(true, {}, defaults, options);

            // 使用 application/json 的方式进行请求时，需要处理相关参数
            if (settings.jsonRequestBody) {
                if (settings.data._method === 'PUT') {
                    settings.httpMethod = 'PUT';
                } else if (settings.data._method === 'DELETE') {
                    settings.httpMethod = 'DELETE';
                }

                delete settings.data._method; // 没必要传递一个无用的参数
                settings.contentType = 'application/json;charset=UTF-8';

                // 非 GET 时 json 对象需要序列化
                if (settings.data.httpMethod !== 'GET') {
                    settings.data = JSON.stringify(settings.data);
                }
            }

            // 替换路径中的变量，例如 /rest/users/{id}, 其中 {id} 为要被 settings.urlParams.id 替换的部分
            if (settings.urlParams) {
                settings.url = settings.url.replace(/\{\{|\}\}|\{(\w+)\}/g, function(m, n) {
                    // m 是正则中捕捉的组 $0，n 是 $1，function($0, $1, $2, ...)
                    if (m == '{{') { return '{'; }
                    if (m == '}}') { return '}'; }
                    return settings.urlParams[n];
                });
            }

            // 执行 AJAX 请求
            $.ajax({
                url        : settings.url,
                data       : settings.data,
                async      : settings.async,
                type       : settings.httpMethod,
                dataType   : 'json',
                contentType: settings.contentType,
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

## 服务器端

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
    /**
     * REST 读取
     * URL: http://localhost:8080/rest
     * 参数: name
     *
     * @param name
     * @return
     */
    @GetMapping("/rest")
    @ResponseBody
    public Result restGet(@RequestParam String name) {
        return Result.ok("GET handled", name);
    }

    /**
     * REST 创建
     * URL: http://localhost:8080/rest
     * 参数: 无
     *
     * @return
     */
    @PostMapping("/rest")
    @ResponseBody
    public Result restPost() {
        return new Result(true, "CREATE handled");
    }

    /**
     * REST 的更新
     * URL: http://localhost:8080/rest
     * 参数: name, age
     *
     * @param name
     * @param age
     * @return
     */
    @PutMapping("/rest")
    @ResponseBody
    public Result restPut(@RequestParam String name, @RequestParam int age) {
        return new Result(true, "UPDATE handled", name + " : " + age);
    }

    /**
     * REST 删除
     * URL: http://localhost:8080/rest
     * 参数: 无
     *
     * @return
     */
    @DeleteMapping("/rest")
    @ResponseBody
    public Result restDelete() {
        return new Result(true, "DELETE handled");
    }

    /**
     * REST 创建，处理 application/json 的请求
     * URL: http://localhost:8080/rest/requestBody
     * 参数: name
     *
     * @return
     */
    @PostMapping("/rest/requestBody")
    @ResponseBody
    public Result restPostJsonRequestBody(@RequestBody String content) {
        return new Result(true, "CREATE requestBody handled: " + content);
    }

    /**
     * REST 更新，处理 application/json 的请求
     * URL: http://localhost:8080/rest/requestBody
     * 参数: name
     *
     * @return
     */
    @PutMapping("/rest/requestBody")
    @ResponseBody
    public Result restUpdateJsonRequestBody(@RequestBody String content) {
        return new Result(true, "UPDATE requestBody handled: " + content);
    }

    /**
     * REST 删除，处理 application/json 的请求
     * URL: http://localhost:8080/rest/requestBody
     * 参数: name
     *
     * @return
     */
    @DeleteMapping("/rest/requestBody")
    @ResponseBody
    public Result restDeleteJsonRequestBody(@RequestBody String content) {
        return new Result(true, "DELETE requestBody handled: " + content);
    }
}
```

## Result.java

Result 用于统一服务器端返回的 JSON 格式，例如:

```json
{
    "code": 0,
    "message": "Short message",
    "success": true,
    "data": {
        "name": "Alice"
    }
}
```

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

## 参考资料

为了理解 Content-Type 为 application/x-www-form-urlencoded 和 application/json 的区别，可以参考四种常见的 POST 提交数据方式 <https://imququ.com/post/four-ways-to-post-data-in-http.html>。