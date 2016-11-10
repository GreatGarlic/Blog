---
title: 绝对坐标布局
date: 2016-11-10 13:16:21
tags: FE
---
如图所使用的布局在管理系统、监控、统计平台以及对话框中经常使用  
![](/img/fe/absolute-layout.png)

主要是使用绝对坐标设置 top, left, right, bottom 和高宽来布局。

<!--more-->

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>Bom 测试</title>
    <script src="http://cdn.bootcss.com/jquery/1.9.1/jquery.min.js" charset="utf-8"></script>

    <style media="screen">
        body {
            font-family: "微软雅黑", Helvetica, Arial, sans-serif;
        }

        .top {
            position: absolute;
            left: 0;
            top: 0;
            right: 0;
            height: 50px;
            background: #FFCF00;
        }

        .left {
            position: absolute;
            left: 0;
            top: 50px;
            bottom: 50px;
            width: 100px;
            background: #BAF300;
        }

        .bottom {
            position: absolute;
            left: 0;
            right: 0;
            bottom: 0;
            height: 50px;
            background: #FF7A00;
        }

        .main {
            position: absolute;
            left: 100px;
            top: 50px;
            right: 0;
            bottom: 50px;
            background: #EFEFEF;
            overflow: auto;
        }
    </style>
</head>

<body>
    <div class="top">Top</div>
    <div class="left">Left</div>
    <div class="main">Main</div>
    <div class="bottom">Bottom</div>

    <script>
        $(document).ready(function() {
            for (var i = 0; i < 100; ++i)
            $('.main').append('Foo<br/>');
        });
    </script>
</body>

</html>
```
