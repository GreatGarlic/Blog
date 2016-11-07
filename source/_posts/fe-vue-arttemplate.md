---
title: Vue-ArtTemplate-jQuery 一起使用
date: 2016-11-07 20:11:05
tags: FE
---
Vue 和 ArtTemplate 绑定数据的格式都是一样的 `{ { } }`，但是 Vue 有个缺点，例如使用类选择器选择 element 时，只有第一个匹配的 element 会生效，还有如树的节点是 Ajax 动态加载的话，用 Vue 也不能多级的显示节点，而这些恰恰 ArtTemplate 能做到，当然 ArtTemplate 不是 MVVM 模式的，所以 Vue 的优点也是 ArtTemplate 不能比的，所以把它们一起结合起来灵活使用看起来不错。

<!--more-->

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>测试</title>
    <script src="http://cdn.bootcss.com/jquery/1.9.1/jquery.min.js" charset="utf-8"></script>
    <script src="http://cdn.staticfile.org/vue/2.0.3/vue.min.js"></script>
    <script src="http://cdn.staticfile.org/artTemplate.js/3.0/template.js" charset="utf-8"></script>

    <style media="screen">
        body {
            font-family: "微软雅黑", "HelveticaNeue-Light", "Helvetica Neue Light", "Helvetica Neue", Helvetica, Arial, sans-serif;
        }
    </style>
</head>

<body>
    <!-- Vue 使用的模版 -->
    <div id="vue-app">
        <button v-on:click="reverseMessage">Reverse</button>
        {{message}}
    </div>
    
    <button id="tl-button">使用 ArtTemplate 模版</button>

    <!-- ArtTemplate 使用的模版 -->
    <script id="tl-foo" type="text/template">
        {{message}}
    </script>

    <script>
        $(document).ready(function() {
            // 使用 Vue
            var app = new Vue({
                el: '#vue-app',
                data: {
                    message: 'What a fucking day!'
                },
                methods: {
                    reverseMessage: function() {
                        this.message = this.message.split(' ').reverse().join(' ');
                    }
                }
            });

            $('#tl-button').click(function(event) {
                // 使用 ArtTemplate
                $('#vue-app').append(template('tl-foo', {
                    message: 'Hello'
                }));
            });
        });
    </script>
</body>

</html>
```
