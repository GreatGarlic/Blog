---
title: 弹出层 Layer
date: 2016-11-09 10:07:41
tags: FE
---
Layer 是一款近年来备受青睐的 web 弹层组件，她具备全方位的解决方案，致力于服务各水平段的开发人员，您的页面会轻松地拥有丰富友好的操作体验，只依赖于 jQuery，提供了多种的弹出层选择，对 iframe 支持友好，可去其官网体验一下具体的例子 <http://layer.layui.com>

> * LeanModal 是一个很简单的弹出层，虽然小，但是太过简陋
> * 使用 Bootstrap 的时候，BootstrapDialog 是不错的选择，但是 Layer 还提供了 tips，msg 等特有的弹出层方式，更简单的自定义等。

![](/img/fe/layer.png)

<!--more-->

使用 Layer，只需要引入 `jquery.js` 和 `layer.js` 即可，相关依赖的 CSS，图片都会自动加载，好方便

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>Bom 测试</title>
    <script src="http://cdn.bootcss.com/jquery/1.9.1/jquery.min.js" charset="utf-8"></script>
    <script src="http://cdn.staticfile.org/layer/2.3/layer.js" charset="utf-8"></script>

    <style media="screen">
        body {
            font-family: "微软雅黑", Helvetica, Arial, sans-serif;
        }
    </style>
</head>

<body>
    <button id="test1">小小提示层</button>
    <div id="box" style="display: none">
        <p>弹出对话框中的自定义内容</p>
        <button id="tip">弹出 Tip</button>
    </div>

    <script>
        $(document).ready(function() {
            $('#test1').on('click', function() {
                layer.open({
                    type: 1,
                    title: '弹出层',              // [可选]
                    content: $('#box'),          // 对话框中的内容部分
                    area: ['600px', '300px'],    // 对话框的大小 [可选]
                    // skin: 'layui-layer-rim',  // 加上边框 [可选]
                    shadeClose: false,           // 为 true 时点击遮罩关闭 [可选]
                    btn: ['确定', '贪婪', '取消'], // 自定义按钮，默认事件为关闭对话框，可以对不同的按钮提供事件处理函数 [可选]
                    yes: function() {            // [可选]
                        layer.msg('啥也不干');    // 确定按钮事件处理
                    },
                    btn2: function() {           // [可选]
                        layer.msg('玩命提示中');
                        return false; // 不返回 false 则对话框会被关闭
                    }
                    // btn3, btn4
                });
            });

            $('#tip').click(function(event) {
                layer.tips('Hi', '#test1');
            });
        });
    </script>
</body>

</html>
```
