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
                window.layerIndex = layer.open({
                    type: 1,
                    title: '弹出层',              // [可选]
                    // closeBtn: 0,              // 不显示关闭按钮
                    // maxWidth: 1000,           // 最大宽度，默认为 360
                    content: $('#box'),          // 对话框中的内容部分
                    area: ['600px', '300px'],    // [可选] 对话框的大小，如果不填，则使用 #box 的大小
                    // skin: 'layui-layer-rim',  // [可选] 加上边框
                    shadeClose: false,           // [可选] 为 true 时点击遮罩关闭
                    btn: ['确定', '贪婪', '取消'], // 自定义按钮，btn1, btn2, btn3
                    btn1: function() {           // [可选] btn1 也可以用 yes
                        layer.msg('啥也不干');    // 确定按钮事件处理
                        layer.close(window.layerIndex); // 调用关闭弹出对话框
                    },
                    btn2: function() {           // [可选]
                        layer.msg('玩命提示中');
                        return false; // 不返回 false 则对话框会被关闭
                    },
                    // btn3, btn4
                    success: function() {}, // 弹出对话框时的回调函数，在此进行初始化
                    end: function() {},     // 关闭对话框时的回调函数，在此进行清理工作
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
