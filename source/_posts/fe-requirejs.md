---
title: 用 RequireJS 统一管理 JS 和 CSS
date: 2017-04-06 17:52:23
tags: FE
---

如果页面很多，每个页面里都使用 `<script> <link>` 来加载 JS, CSS，就会显得很分散，开发环境和生产环境中想要修改多处 JS, CSS 的路径时不够方便，例如生产环境使用七牛等 CDN 加载 jQuery，而测试环境不能访问外网，这时 CDN 就玩了（不要问我为什么，就是有人这么干，我被坑的不要不要的），只能把 jQuery 放到本地了，为此需要到所有页面里修改代码然后测试，很不方便。使用 RequireJS 后就可以集中的管理 JS, CSS，修改起来比较方便，也可以使用 Build 工具根据不同的环境进行修改。

下面介绍使用 RequireJS 加载 jQuery, Layer, Vue, SemanticUi。

<!--more-->

**项目目录:**

```
├── a.html
├── js
│   └── require-config.js
└── lib
    ├── css.min.js
    ├── jquery.js
    └── require.js
```

> require.js 下载: <https://cdn.bootcss.com/require.js/2.3.3/require.min.js>
>
> css.min.js 下载: <https://github.com/guybedford/require-css>
>
> 这 2 个文件是根，如果还使用 CDN 的话，非 CDN 时就完蛋了。

## RequireJS 的配置文件

```js
// 文件名: require-config.js
require.config({
    paths: {
        jquery:     ['//cdn.bootcss.com/jquery/1.9.1/jquery.min', '/lib/jquery'],
        layer:      '//cdn.staticfile.org/layer/2.3/layer',
        vue:        '//cdn.staticfile.org/vue/2.0.3/vue',
        semanticUi: '//cdn.staticfile.org/semantic-ui/2.2.7/semantic.min'
    },
    shim: {
        layer: {
            deps: ['jquery', 'css!//cdn.staticfile.org/layer/2.3/skin/layer.css']
        },
        semanticUi: {
            deps: ['jquery', 'css!//cdn.staticfile.org/semantic-ui/2.2.7/semantic.min.css']
        }
    },
    map: {
        '*': {
            css: '/lib/css.min.js'
        }
    }
});
```

> require-config.js 中定义所有需要的 JS, CSS 的路径，修改的时候只要修改此文件即可
>
> Layer 依赖 jQuery，同时还有 CSS 需要加载
>
> SemanticUi 依赖 jQuery，同时还有 CSS 需要加载。由于 SemanticUi 的 CSS 是布局和组件使用的，在 DOM 加载后才加载的话页面开始比较乱，然后 SemanticUi 才生效，看上去不是很好，使用 link 加载 SemanticUi 就不会这样了，看需要再决定是否使用 RequireJS 加载它吧。
>
> css.min.js 用来实现加载依赖的 CSS
>
> jQuery 有 2 个下载地址，第一个下载失败会尝试用第二个地址下载，是不是就可以在测试环境里使用本地 jQuery 了？

## 页面文件

```html
<!-- 文件名: a.html -->
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title></title>
</head>

<body>
    <style>
        body {
            padding: 100px;
        }
    </style>

    <!-- 使用 Vue -->
    <div id="vue-app">
        <span v-text="message"></span>
    </div>

    <button id="button" class="ui button">Button</button>

    <!-- 使用 Semantic-Ui -->
    <div class="ui selection dropdown">
        <input type="hidden" name="gender">
        <i class="dropdown icon"></i>
        <div class="default text">Gender</div>
        <div class="menu">
            <div class="item" data-value="male" data-text="Male">
                <i class="male icon"></i> Male
            </div>
            <div class="item" data-value="female" data-text="Female">
                <i class="female icon"></i> Female
            </div>
        </div>
    </div>

    <!-- 使用 RequireJS 加载需要的模块 -->
    <!-- require.js 的 data-main 竟然不生效，坑爹啊，所以只能单独下载配置了 -->
    <script src="/lib/require.js"></script>
    <script src="/js/require-config.js"></script>
    <script>
        require(['jquery', 'layer', 'vue', 'semanticUi'], function($, layer, Vue) {
            $('#button').click(function(event) {
                layer.msg('玩命提示中');
            });

            new Vue({
                el: '#vue-app',
                data: {
                    message: 'Hello'
                }
            });

            // 使用 Semantic-Ui
            $('.ui.dropdown').dropdown();
        });

        require(['layer'], function() {
            // layer 没有再次加载
        });
    </script>
</body>

</html>
```

## 测试

* 访问 <http://localhost/a.html>
* 看到 Layer 生效了，SemanticUi 也生效了，require 了 2 次 layer，没有重复加载

## 参考资料
* [加载不符合AMD规范的 JS 文件](http://blog.csdn.net/ligang2585116/article/details/50725430)
* [使用 RequireJS 的 shim 参数，完成 jQuery 插件的加载](http://www.tuicool.com/articles/vMZBnyr)


