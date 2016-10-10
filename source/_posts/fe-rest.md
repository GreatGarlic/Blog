---
title: jQuery 的 REST 插件
date: 2016-10-10 16:58:14
tags: FE
---
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
$.rest.get({url: '/rest', data: {name: 'Alice'}, done: function(result) {
    console.log(result);
}});
```

<!--more-->

## 测试页面
```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>SpringMVC REST</title>
    <script src="/lib/jquery.min.js" charset="utf-8"></script>
    <script src="/js/jquery.rest.js" charset="utf-8"></script>
</head>

<body>
    <script>
        $(document).ready(function() {
            $.rest.get({url: '/rest', data: {name: 'Alice'}, done: function(result) {
                console.log(result);
            }});

            $.rest.create({url: '/rest', done: function(result) {
                console.log(result);
            }});

            $.rest.update({url: '/rest', data: {name: 'Bob', age: 22}, done: function(result) {
                console.log(result);
            }});

            $.rest.delete({url: '/rest', done: function(result) {
                console.log(result);
            }});
        });
    </script>
</body>

</html>
```

## 输出
![](/img/fe/rest-output.png)

## jquery.rest.js
```js
/**
 * 执行 REST 请求的 jQuery 插件:
 *      Get    请求调用 $.rest.get()
 *      Create 请求调用 $.rest.create()
 *      Update 请求调用 $.rest.update()
 *      Delete 请求调用 $.rest.delete()
 *
 * 调用例子:
 *      $.rest.get({url: '/rest', data: {name: 'Alice'}, done: function(result) {
 *          console.log(result);
 *      }}, fail: function(error) {});
 */
$.rest = {
    /**
     * 使用 Ajax 的方式执行 REST 的 GET 请求(服务器响应的数据根据 REST 的规范，应该是 Json 对象).
     * 以下几个 REST 的函数 $.restCreate(), $.restUpdate(), $.restDelete() 只是请求的 HTTP 方法和 data 处理不一样，其他的都是相同的.
     *
     * @param {[json]} options 有以下几个选项:
     *        {[string]}   url   请求的 URL(必选)
     *        {[json]}     data  请求的参数(可选)
     *        {[function]} done  请求成功时的回调函数(可选)
     *        {[function]} fail  请求失败时的回调函数(可选)
     *        {[function]} alway 请求完成后的回调函数(可选)
     * @return 没有返回值
     */
    get: function(options) {
        options.httpMethod = 'GET';
        this.execute(options);
    },
    create: function(options) {
        options.httpMethod = 'POST';
        options.data = options.data ? JSON.stringify(options.data) : {};
        this.execute(options);
    },
    update: function(options) {
        options.httpMethod = 'PUT';
        options.data = options.data ? JSON.stringify(options.data) : {};
        this.execute(options);
    },
    delete: function(options) {
        options.httpMethod = 'DELETE';
        options.data = options.data ? JSON.stringify(options.data) : {};
        this.execute(options);
    },
    /**
     * 执行 Ajax 请求，不推荐直接调用这个方法.
     * 因为发送给 SpringMVC 的 Ajax 请求中必须满足下面的条件才能执行成功:
     *     1. dataType: 'json'
     *     2. contentType: 'application/json'
     *     3. data: GET 时为 json 对象，POST, PUT, DELETE 时必须为 JSON.stringify(data)
     *
     * @param {[json]} options 有以下几个选项:
     *        {[string]}   url        请求的 URL(必选)
     *        {[string]}   httpMethod 请求的方式，有 GET, PUT, POST, DELETE(必选)
     *        {[json]}     data       请求的参数(可选)
     *        {[function]} done       请求成功时的回调函数(可选)
     *        {[function]} fail       请求失败时的回调函数(可选)
     *        {[function]} alway      请求完成后的回调函数(可选)
     */
    execute: function(options) {
        var defaults = {
            data: {},
            done: function() {},
            fail: function() {},
            always: function() {}
        };

        var settings = $.extend({}, defaults, options);

        $.ajax({
            url:  settings.url,
            data: settings.data,
            type: settings.httpMethod,
            dataType: 'json',
            contentType: 'application/json'
        })
        .done(function(result) {
            settings.done(result);
        })
        .fail(function(error) {
            settings.fail(error);
        })
        .always(function() {
            settings.always();
        });
    }
};
```

## 服务器端代码
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
    public Result handleGet(@RequestParam String name) {
        return new Result(true, "GET handled", name);
    }

    @PutMapping("/rest")
    @ResponseBody
    public Result handlePut(@RequestBody Map map) {
        return new Result(true, "UPDATE handled", map);
    }

    @PostMapping("/rest")
    @ResponseBody
    public Result handlePost() {
        return new Result(true, "CREATE handled");
    }

    @DeleteMapping("/rest")
    @ResponseBody
    public Result handleDelete() {
        return new Result(true, "DELETE handled");
    }
}
```

```java
package com.xtuer.bean;

public class Result {
    private boolean success;
    private String message;
    private Object data;

    public Result(boolean success, String message) {
        this(success, message, null);
    }

    public Result(boolean success, String message, Object data) {
        this.success = success;
        this.message = message;
        this.data = data;
    }

    public boolean isSuccess() {
        return success;
    }

    public void setSuccess(boolean success) {
        this.success = success;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public Object getData() {
        return data;
    }

    public void setData(Object data) {
        this.data = data;
    }
}
```
