---
title: JS 绘制椭圆
date: 2017-06-24 22:15:18
tags: FE
---
Canvas 还没有提供直接绘制椭圆的功能，下面使用 bezierCurveTo() 来绘制椭圆。

> 圆也是使用 arc 来绘制的，在新版的 JS 中提供了 ellipse 来绘制椭圆，但是很多浏览器都还不支持

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <script src="http://cdn.bootcss.com/jquery/3.2.1/jquery.min.js"></script>
    <style media="screen">
        canvas {
            border: 1px solid grey;
        }
    </style>
</head>

<body>
    <canvas id="canvas" width="500" height="300">Your browser does not support canvas.</canvas>

    <script>
        var canvas = $('#canvas').get(0);
        var ctx = canvas.getContext('2d');

        drawEllipse(ctx, 100, 100, 80, 120);
        drawEllipse(ctx, 200, 200, 200, 80);

        function drawEllipse(context, centerX, centerY, width, height) {
            context.beginPath();
            context.moveTo(centerX, centerY - height / 2);

            context.bezierCurveTo(
                centerX + width / 2, centerY - height / 2,
                centerX + width / 2, centerY + height / 2,
                centerX, centerY + height / 2
            );
            context.bezierCurveTo(
                centerX - width / 2, centerY + height / 2,
                centerX - width / 2, centerY - height / 2,
                centerX, centerY - height / 2
            );
            context.closePath();
            context.stroke();
        }
    </script>
</body>

</html>
```

