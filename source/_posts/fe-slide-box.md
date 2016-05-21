---
title: jQuery Slide Box
date: 2016-05-21 18:31:00
tags: FE
---
如下图作为导航时，同时只有一个部分能够展开，例如点击 `访问数据库` 则展开 `访问数据库` 并收缩 `QSS`

![](/img/fe/slide-box.png)

<!--more-->

## 核心代码

```js
$(document).ready(function() {
    // 非第一个 ul 下的 li 隐藏
    $('#slide-box ul:gt(0) li').hide();

    // 点击 ul>a 展开点击的 ul，并隐藏其他 ul 下的 li
    $('#slide-box ul>a').click(function(e) {
        e.preventDefault();
        $(this).parent().siblings().find('li').slideUp();
        $(this).parent().find('li').slideDown();
    });
});
```

## 全部代码
```html
<!DOCTYPE html>
<html>
<head>
    <title>Slide Box</title>
    <meta charset="UTF-8">
    <style>
    #slide-box {
        width: 256px;
        border: 1px solid rgba(0,0,0,0.7);
    }

    #slide-box ul > a {
        display: block;
        color: #EEE;
        background: rgba(0,0,0,0.7);
        text-decoration: none;
        padding: 2px 5px;
    }

    #slide-box ul {
        padding-left: 0;
        margin-top: 0;
    }

    #slide-box ul:last-child {
        margin-bottom: 0;
        padding-bottom: 15px;
    }

    #slide-box li {
        margin-left: 20px;
        list-style: none;
    }
    </style>
</head>
<body>
    <div id="slide-box">
        <ul>
            <a href="#">Qt 绘图</a>
            <li><a href="#">绘图基础</a></li>
            <li><a href="#">用画家的思维绘制图形</a></li>
            <li><a href="#">绘制平滑曲线</a></li>
            <li><a href="#">实时动态曲线</a></li>
        </ul>
        <ul>
            <a href="#">QSS</a>
            <li><a href="#">QSS 基础</a></li>
            <li><a href="#">加载 QSS</a></li>
            <li><a href="#">盒子模型</a></li>
            <li><a href="#">QSS 选择器</a></li>
        </ul>
        <ul>
            <a href="#">访问数据库</a>
            <li><a href="#">数据库驱动</a></li>
            <li><a href="#">访问 SQLite</a></li>
            <li><a href="#">访问 MySql</a></li>
        </ul>
        <ul>
            <a href="#">动画</a>
            <li><a href="#">QPropertyAnimation</a></li>
            <li><a href="#">动画插值函数与高级运用</a></li>
            <li><a href="#">传统动画实现、缓冲动画原理</a></li>
            <li><a href="#">状态机与Qt动画一起使用</a></li>
        </ul>
    </div>

    <script src="/js/jquery.min.js"></script>
    <script>
    $(document).ready(function() {
        // 非第一个 ul 下的 li 隐藏
        $('#slide-box ul:gt(0) li').hide();

        // 点击 ul>a 展开点击的 ul，并隐藏其他 ul 下的 li
        $('#slide-box ul>a').click(function(e) {
            e.preventDefault();
            $(this).parent().siblings().find('li').slideUp();
            $(this).parent().find('li').slideDown();
        });
    });
    </script>
</body>
</html>
```
