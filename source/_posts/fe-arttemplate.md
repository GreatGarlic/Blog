---
title: JS 模版
date: 2016-07-14 07:54:44
tags: FE
---

JS 中有很多模版库，例如 `ArtTemplate`、doT、laytpl、Mustache、Underscore Templates、Embedded JS Templates、HandlebarsJS、Jade templating、baiduTemplate、kissyTemplate 等等，总的比较下来，语法上更喜欢的是 `ArtTemplate`，其性能也不错；`laytpl` 号称性能之王，比 ArtTemplate 和 doT 还快，如果追求性能，可以试试它 <http://laytpl.layui.com>。

<!--more-->

## ArtTemplate 用法:

1. 定义模版

    ```js
    // type="text/html" 是为了让其不备浏览器作为 JS 解析，同时还不可见
    <script id="templateId" type="text/html">
        Your name: {{name}}
    </script>
    ```
2. 定义模版使用的 Json 对象:

    ```js
    var data = {name: 'John'};
    ```
3. 使用 `函数 template() + 模版 + 数据` 生成 HTML

    ```js
    // templateId 是 <script> 的 id
    var html = template('templateId', data);
    ```
4. 添加生成的 HTML 到 DOM

## ArtTemplate 实例

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>Art Template Demo</title>
    <script src="template.js" charset="utf-8"></script>
    <script src="jquery.min.js" charset="utf-8"></script>
</head>

<body>
    <div id="zoo">
        <!-- [1]. 定义模版 -->
        <script id="templateId" type="text/html">
            <ul data-name="{{name}}" data-city="{{city}}">
                <!--遍历数组-->
                {{each animals as animal}}
                    <li>{{animal}}</li>
                {{/each}}
            </ul>
        </script>
    </div>

    <script type="text/javascript">
        $(document).ready(function() {
            // [2]. 模版使用的数据
            var data = {
                name: 'Zoologischer Garten',
                city: 'Berlin',
                animals: ['大象', '老虎', '狮子', '猴子']
            };

            // [3]. 使用模版生成 HTML，默认对输出进行转义，不转义使用 {{#value}}
            var html = template('templateId', data);
            $('#zoo').append(html); // [4]. 添加生成的 HTML 到 DOM
        });
    </script>
</body>

</html>
```

## ArtTemplate Github
更多的详细说明和使用案例请参考 <https://github.com/aui/artTemplate>
