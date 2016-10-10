---
title: 使用 REST 的风格封装 jQuery.ajax()
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

如果每个 REST 的请求都像上面这样写一遍: `PUT`, `POST`, `DELETE` 时需要 `JSON.stringify(data)`, 请求不同时 type 也不同，dataType 和 contentType 是固定的，这么多限制，很容易出错，我们可以对其作一个简单的封装(请看下面的 `rest.js`)，然后就使用更有语义的函数进行 RESTful 风格的访问了，而且不容易发生错误:

```js
Rest.update('/users/1/username', {name: 'Bob'}, function(result) {
    console.log(result);
});
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
    <script src="/js/rest.js" charset="utf-8"></script>
</head>

<body>
    <script>
        $(document).ready(function() {
            Rest.get('/rest', {name: 'Alice'}, function(result) {
                console.log(result);
            });

            Rest.update('/rest', {name: 'Bob', age: 22}, function(result) {
                console.log(result);
            });

            Rest.create('/rest', {}, function(result) {
                console.log(result);
            });

            Rest.delete('/rest', {}, function(result) {
                console.log(result);
            });
        });
    </script>
</body>

</html>
```

## 输出
![](/img/fe/rest-output.png)

## rest.js
```js
/**
 * REST 工具类，使用 Ajax 执行 REST 请求
 */
function Rest() {
}

/**
 * 使用 Ajax 的方式执行 REST 的 GET 操作，并且表明服务器响应的数据格式是 JSON 格式.
 * [以下几个 REST 的函数 create, update, delete 只是请求的 HTTP 方法和 data 处理不一样，其他的都是相同的]
 *
 * @param  {[string]}   url              请求的 URL
 * @param  {[json]}     data             请求参数的 Json 对象
 * @param  {[string]}   httpMethod       请求的方法，为 'GET', 'PUT'(更新), 'POST'(创建), 'DELETE'
 * @param  {[function]} successCallback  请求成功时的回调函数
 * @param  {[function]} failCallback     请求失败时的回调函数
 * @param  {[function]} completeCallback 请求完成后的回调函数
 * @return 没有返回值
 */
Rest.get = function(url, data, successCallback, failCallback, completeCallback) {
    Rest.ajax(url, data, 'GET', successCallback, failCallback, completeCallback);
};

Rest.create = function(url, data, successCallback, failCallback, completeCallback) {
    Rest.ajax(url, JSON.stringify(data), 'POST', successCallback, failCallback, completeCallback);
};

Rest.update = function(url, data, successCallback, failCallback, completeCallback) {
    Rest.ajax(url, JSON.stringify(data), 'PUT', successCallback, failCallback, completeCallback);
};

Rest.delete = function(url, data, successCallback, failCallback, completeCallback) {
    Rest.ajax(url, JSON.stringify(data), 'DELETE', successCallback, failCallback, completeCallback);
};

/**
 * 执行 Ajax 请求.
 * 因为发送给 SpringMVC 的 Ajax 请求中必须满足下面的条件才能执行成功:
 *     1. dataType: 'json'
 *     2. contentType: 'application/json'
 *     3. data: GET 是为 json 对象，POST, PUT, DELETE 时必须为 JSON.stringify(data)
 * 所以把执行 Ajax 请求的函数提取出来作为一个函数，以免不小心出错.
 *
 * @param  {[string]}   url              请求的 URL
 * @param  {[json]}     data             请求参数的 Json 对象
 * @param  {[string]}   httpMethod       请求的方法，为 'GET', 'PUT'(更新), 'POST'(创建), 'DELETE'
 * @param  {[function]} successCallback  请求成功时的回调函数
 * @param  {[function]} failCallback     请求失败时的回调函数
 * @param  {[function]} completeCallback 请求完成后的回调函数
 * @return 没有返回值
 */
Rest.ajax = function(url, data, httpMethod, successCallback, failCallback, completeCallback) {
    $.ajax({
        url: url,
        data: data,
        type: httpMethod,
        dataType: 'json',
        contentType: 'application/json'
    })
    .done(function(result) {
        if ($.isFunction(successCallback)) {
            successCallback(result);
        }
    })
    .fail(function(response) {
        if ($.isFunction(failCallback)) {
            failCallback(response);
        }
    })
    .always(function() {
        if ($.isFunction(completeCallback)) {
            completeCallback();
        }
    });
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
