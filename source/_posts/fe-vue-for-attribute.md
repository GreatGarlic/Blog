---
title: Vue 使用 v-for 和设置属性及事件处理
date: 2016-11-08 13:48:19
tags: [FE, Vue]
---

Vue 中使用 `v-for` 遍历数组，设置属性使用指令 `v-bind`，输入和数据绑定用 `v-model`，事件处理则用 `v-on`。

插值时需要用 `{ { } }`，但是使用指令时不用，指令用 `v-` 作为前缀，例如

* v-for
* v-if
* v-show
* v-bind
* v-on

v-bind 和 v-on 有简写形式 `:` 和 `@`

* 例如设置属性 `data-name` 使用指令 `v-bind:data-name`，其简写为 `:data-name`
* 点击事件使用 `v-on:click`，其简写为 `@click`

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
            font-family: "微软雅黑", Helvetica, Arial, sans-serif;
        }

        .active {
            background: #EEE;
        }

        li {
            cursor: default;
        }
    </style>
</head>

<body>
    <!-- Vue 使用的模版 -->
    <div id="vue-app">
        <input type="text" v-model="username" @keyup.enter="alert(username)" placeholder="输入用户名">
        
        <ul>
            <li v-for="province in provinces" :data-id="province.id">{{province.name}}</li>
            <li v-show="provinces.length === 0">没有数据</li>
        </ul>
        <hr/>
        <ul>
            <li v-for="(province, index) in provinces"
                :data-id="province.id"
                :class="currentIndex==index? 'active' : ''"
                @click="currentIndex=index">
                {{province.name}} - {{index}}
            </li>
        </ul>

        Current Index: {{currentIndex}}
    </div>

    <script>
        $(document).ready(function() {
            // 使用 Vue
            var app = new Vue({
                el: '#vue-app',
                data: {
                    username: '',
                    currentIndex: -1,
                    provinces: [
                        {id: 100, name: '北京'},
                        {id: 102, name: '上海'},
                        {id: 103, name: '深圳'},
                        {id: 104, name: '贵州'}
                    ]
                }
            });
        });
    </script>
</body>

</html>
```

## 参考资料
[官方文档](https://cn.vuejs.org/v2/guide/forms.html)
