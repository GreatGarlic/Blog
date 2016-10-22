---
title: 时间选择器 Laydate
date: 2016-10-21 22:26:09
tags: FE
---
LayDate 包含了

* 日期范围限制
* 开始日期设定
* 自定义日期格式
* 时间戳转换
* 当天的前后若干天返回
* 时分秒选择
* 智能响应
* 自动纠错
* 节日识别
* 快捷键操作

<!--more-->

LayDate 致力于成为全球最用心的 web 日期支撑，为国内外所有从事 web 应用开发的同仁提供力所能及的动力。她基于原生 JavaScript 精心雕琢，兼容了包括 IE6 在内的所有主流浏览器。她具备优雅的内部代码，良好的性能体验，和完善的皮肤体系，并且完全开源，你可以任意获取开发版源代码，一扫某些传统日期控件的封闭与狭隘。LayDate 本着资源共享的开发者精神和对网页日历交互无穷的追求，延续了 Layui 一贯的简单与易用。她遵循 LGPL 协议，您可以免费将她用于任何个人项目。

只需要引入 `laydate.js` 即可，最简单的使用可以如下，点击 input 就会弹出时间选择器

```html
<input onclick="laydate()">
```

更多例子和文档请访问 <http://laydate.layui.com>

附上一个简单完整的例子

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>日期选择器</title>
    <script src="laydate/laydate.js" charset="utf-8"></script>
</head>

<body>

    开始日: <input id="start" class="laydate-icon"/>
    结束日: <input id="end" class="laydate-icon"/>
    <script>
        var start = {
            elem: '#start',
            format: 'YYYY-MM-DD',
            istoday: true
        };

        var end = {
            elem: '#end',
            format: 'YYYY-MM-DD',
            istoday: true
        };

        laydate(start);
        laydate(end);
    </script>
</body>

</html>
```
