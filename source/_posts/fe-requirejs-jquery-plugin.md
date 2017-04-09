---
title: RequireJS 加载 jQuery 插件
date: 2017-04-09 08:06:03
tags: FE
---

jQuery 插件使用 RequireJS 加载，只需要在 shim 中配置插件依赖 jQuery 就可以了，下面以加载 SemanticUi 为例。<!--more-->

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>加载 jQuery 插件</title>
    <link rel="stylesheet" href="//cdn.staticfile.org/semantic-ui/2.2.7/semantic.min.css">
</head>

<body>
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

    <script src="/lib/require.js"></script>
    <script>
        require.config({
            paths: {
                jquery: ['//cdn.bootcss.com/jquery/1.9.1/jquery.min', '/lib/jquery'],
                semanticUi: '//cdn.staticfile.org/semantic-ui/2.2.7/semantic.min'
            },
            shim: {
                semanticUi: {
                    deps: ['jquery']
                }
            }
        });
    </script>
    <script>
        require(['jquery', 'semanticUi'], function($) {
            $('.dropdown').dropdown();
        });
    </script>
</body>

</html>
```

