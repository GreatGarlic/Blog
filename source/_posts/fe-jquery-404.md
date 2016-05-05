---
title: jQuery 404 时调用的方法
date: 2016-05-04 20:28:07
tags: FE
---

jQuery 的 `ajax` 能给不同的响应状态码指定回调函数，例如当连不上服务器(服务器没启动，网络有问题等)时调用 `statusCode` 中 `0` 的方法，连上了服务器，但是找不到要访问的 URL 则调用 `statusCode` 中 `404` 的方法。

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
                404: function() { // 找不到指定的 URL
                    console.log('404: Page not found');
                },
                0: function() { // 连不上服务器
                    console.log('0: Server is not started');
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
