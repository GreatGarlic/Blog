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

同样的效果，使用 flex 布局如下，简单很多:

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title></title>
    <style media="screen">
        html,
        body {
            display: flex;
            flex-direction: column;
            padding: 0;
            margin: 0;
            height: 100%;
            font-family: "微软雅黑", Helvetica, Arial, sans-serif;
        }

        .header {
            height: 80px;
            background: #FFCF00;
            flex-shrink: 0;
        }

        .footer {
            height: 80px;
            background: #FF7A00;
            flex-shrink: 0;
        }

        .main {
            flex: auto;
            display: flex;
            flex-direction: row;
        }

        .sidebar {
            width: 150px;
            background: #BAF300;
        }

        .content {
            flex: auto;
            background: white;
            overflow: auto;
        }
    </style>
</head>

<body>
    <div class="header"></div>
    <div class="main">
        <div class="sidebar"></div>
        <div class="content"></div>
    </div>
    <div class="footer"></div>

    <script src="http://cdn.bootcss.com/jquery/1.9.1/jquery.min.js" charset="utf-8"></script>
    <script type="text/javascript">
        $(document).ready(function() {
            for (var i = 0; i < 100; ++i) {
                $('.content').append('Foo<br/>');
            }
        });
    </script>
</body>

</html>
```

