---
title: Semantic Ui 的侧边栏
date: 2017-03-14 09:46:15
tags: FE
---

侧边栏在页面打开后一直存在，并且间隔和内容区域的间隔不要太大，可以使用 CSS 调整一下  pusher 的 translate3d 的位置，icon menu 的文本都是居中的，如果不想居中，也需要自己调整，效果如下

![](/img/fe/semantic-ui-sidebar.png)

<!--more-->

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title></title>
    <script src="http://cdn.bootcss.com/jquery/3.1.1/jquery.min.js"></script>
    <script src="http://cdn.staticfile.org/semantic-ui/2.2.7/semantic.min.js"></script>
    <link href="http://cdn.staticfile.org/semantic-ui/2.2.7/semantic.css" rel="stylesheet">

    <style media="screen">
        body {
            padding: 20px;
        }

        /* 使得子菜单的文本左对齐 */
        .item-menu .header, .item-menu .item {
            text-align: left !important;
        }

        /* 内容部分离菜单更近 */
        .pusher {
            transform: translate3d(103px, 0, 0) !important;
        }
    </style>
</head>

<body>
    <!-- 菜单 -->
    <div class="ui left vertical inverted visible sidebar labeled icon menu">
        <a class="item"><i class="home icon"></i>Home</a>
        <a class="item"><i class="block layout icon"></i> Topics</a>
        <a class="item"><i class="smile icon"></i> Friends</a>
        <div class="item item-menu">
            <div class="header">Collections</div>
            <div class="menu">
                <a class="item" href="/collections/breadcrumb.html">Breadcrumb</a>
                <a class="item" href="/collections/form.html">Form</a>
                <a class="item" href="/collections/grid.html">Grid</a>
                <a class="item" href="/collections/menu.html">Menu</a>
            </div>
        </div>
    </div>

    <!-- 网页的主要部分 -->
    <div class="pusher">
        <div class="ui card">
            <div class="image">
                <img src="http://omqpd0pt4.bkt.clouddn.com/ade.jpg">
            </div>
            <div class="content">
                <a class="header">Kristy</a>
                <div class="meta">
                    <span class="date">Joined in 2013</span>
                </div>
                <div class="description">
                    Kristy is an art director living in New York.
                </div>
            </div>
            <div class="extra content">
                <a><i class="user icon"></i> 22 Friends</a>
            </div>
        </div>
    </div>
</body>

</html>
```

更多内容请参考 [Semantic-Ui Sidebar](https://semantic-ui.com/modules/sidebar.html#/definition)