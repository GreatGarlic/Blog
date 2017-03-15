---
title: Semantic Ui 菜单栏
date: 2017-03-14 15:39:56
tags: FE
---

使用 menu 创建菜单栏:

![](/img/fe/semantic-ui-menu.png)

> 注意和 siderbar menu 的区别

<!--more-->

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title></title>
    <script src="http://cdn.bootcss.com/jquery/3.1.1/jquery.min.js"></script>
    <script src="http://cdn.staticfile.org/semantic-ui/2.2.7/semantic.min.js"></script>
    <link href="http://cdn.staticfile.org/semantic-ui/2.2.7/semantic.min.css" rel="stylesheet">

    <style media="screen">
        body {
            padding: 20px;
        }
    </style>

    <script>
        $(document).ready(function() {
            // 鼠标放到 dropdown 时显示下拉菜单，默认只有点击后才显示
            $('.dropdown.item').dropdown({
                on: 'hover'
            });
        });
    </script>
</head>

<body>
    <div class="ui inverted menu">
        <div class="header item">Brand</div>
        <div class="active item">Home</div>
        <a class="item">Vitae</a>
        <div class="ui dropdown item" tabindex="0">
            Dropdown<i class="dropdown icon"></i>
            <div class="menu" tabindex="-1">
                <div class="item">Action</div>
                <div class="item">Another Action</div>
                <div class="item">Something else here</div>
                <div class="divider"></div>
                <div class="item">Separated Link</div>
                <div class="divider"></div>
                <div class="item">One more separated link</div>
            </div>
        </div>
        <div class="right menu">
            <div class="item">
                <div class="ui transparent inverted icon input">
                    <i class="search icon"></i>
                    <input type="text" placeholder="Search">
                </div>
            </div>
            <a class="item">Login</a>
        </div>
    </div>
</body>

</html>
```

