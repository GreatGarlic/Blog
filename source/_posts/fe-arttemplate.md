---
title: JS 模版 artTempalte
date: 2016-07-14 07:54:44
tags: FE
---
使用模版就可以不用在 JS 里手动的使用字符串相加拼写 HTML 片段了。

JS 中有很多模版库，例如 `artTempalte `、doT、juicer、laytpl、Mustache、Underscore Templates、Embedded JS Templates、HandlebarsJS、Jade templating、baiduTemplate、kissyTemplate 等等，总的比较下来，语法上更喜欢的是 `artTempalte `，其性能也不错；`laytpl` 号称性能之王，比 artTemplate 和 doT 还快，如果追求性能，可以试试它 <http://laytpl.layui.com>。

<!--more-->

## artTempalte 用法:

1. 定义模版

    ```js
    // type="text/html" 是为了让其不被浏览器作为 JS 解析，同时还不可见
    <script id="templateId" type="text/html">
        Your name: {{name}}
    </script>
    ```
2. 定义模版使用的数据的 Json 对象:

    ```js
    var data = {name: 'John'};
    ```
3. 使用 `函数 template() + 模版 + 数据` 生成 HTML

    ```js
    // templateId 是 <script> 的 id
    var html = template('templateId', data);
    ```
4. 添加生成的 HTML 到 DOM

## artTempalte 实例

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>Art Template Demo</title>
    <script src="template.js" charset="utf-8"></script>
</head>

<body>
    <div id="zoo">
        <!-- [1]. 定义模版 -->
        <script id="templateId" type="text/html">
            <ul data-name="{{name}}" data-city="{{city}}">
                <!--遍历基础类型数组-->
                {{each animals as animal}}
                <li>{{animal}}</li>
                {{/each}}
            </ul>

            <ul>
                <!--遍历对象数组-->
                {{each staffs as staff}}
                <li>{{staff.name}} - {{staff.age}}</li>
                {{/each}}
            </ul>
        </script>
    </div>

    <script type="text/javascript">
        // [2]. 模版使用的数据
        var data = {
            name: 'Zoologischer Garten',
            city: 'Berlin',
            animals: ['大象', '老虎', '狮子', '猴子'], // 基础类型数组
            staffs: [{ name: 'Alice', age: 20 }, { name: 'Bob', age: 22 }] // 对象数组
        };

        // [3]. 使用模版生成 HTML，默认对输出进行转义，不转义使用 {{#value}}
        var html = template('templateId', data);
        
        // [4]. 添加生成的 HTML 到 DOM
        document.getElementById('zoo').innerHTML = html; 
    </script>
</body>

</html>
```

## ArtTemplate Github
更多的详细说明和使用案例请参考 <https://github.com/aui/artTemplate>

下载 artTemplate: <https://raw.github.com/aui/artTemplate/master/dist/template.js>
