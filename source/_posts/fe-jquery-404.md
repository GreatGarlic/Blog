---
title: jQuery 404 时调用的方法
date: 2016-05-04 20:28:07
tags: FE
---

当连不上服务器(服务器没启动，网络有问题等)，或者连上了服务器，但是要访问的 URL 在服务器上没有提供(这时报 404 错误)，都会调用 `statusCode` 中 `404` 的方法。

<!--more-->

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <script src="http://libs.baidu.com/jquery/1.11.1/jquery.min.js"></script>
    <script>
    $(document).ready(function() {
        $.ajax({
            url: '/path/to/file',
            type: 'GET',
            dataType: 'json',
            statusCode: {
                404: function() { // 连不上服务器，找不到页面都会调用这个方法
                    console.log( "404: page not found" );
                }
            }
        })
        .done(function() {
            console.log("success");
        })
        .fail(function() {
            console.log("error");
        })
        .always(function() {
            console.log("complete");
        });
    });
    </script>
</head>

<body>
</body>
</html>
```
