---
title: Semantic Ui Tab
date: 2017-03-15 10:44:40
tags: [FE, Semantic-Ui]
---

Semantic Ui 中创建 Tab，需要 2 步:

1. 定义 Tab 的 HTML 结构

   关键是 **.item** 和 **.tab** 中的属性 **data-tab** 的定义，用于关联跳转

   有 class **active** 的 .item 和 .tab 是当前的 Tab，其他的隐藏

2. JS 实现点击切换 Tab

![](/img/fe/semantic-ui-tab.png)

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

        p {
            height: 50px;
            background: #CCC;
        }
    </style>

    <script>
        $(document).ready(function() {
            $('.tabular.menu .item').tab(); // [2] 使能够点击切换 tab
        });
    </script>
</head>

<body>
    <div class="ui container grid">
        <!-- [1] 定义 tab -->
        <div class="ui row">
            <div class="ui top attached tabular menu">
                <div class="item" data-tab="0">Tag</div>
                <div class="item" data-tab="1">User</div>
                <div class="active item" data-tab="2">Example</div>
            </div>
            <div data-tab="0" class="ui bottom attached tab segment"><p></p></div>
            <div data-tab="1" class="ui bottom attached tab segment"><p></p><p></p></div>
            <div data-tab="2" class="ui bottom attached active tab segment"><p></p><p></p><p></p></div>
        </div>

        <div class="stackable two column row">
            <div class="ui violet eight wide column"><p></p></div>
            <div class="ui purple eight wide column"><p></p></div>
        </div>
    </div>
</body>

</html>
```

