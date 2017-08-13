---
title: 截取 Canvas 绘制的图形
date: 2017-08-13 22:14:45
tags: FE
---

使用 canvas 绘图时，很多时候在 canvas 上绘制的图像只占 canvas 的一部分，如果把整个 canvas 的图像发送到服务器就比较浪费空间和带宽，所以保存真正绘制的图像部分是有必要的。

下图的上部分为一个的 canvas，绘制的图像大概只占它的四分之一，我们的目的是把 canvas 中多余的部分去掉，得到真正绘制的图像，如下部分显示:

![](/img/fe/canvas-image-bounding-rect.png)

<!--more-->

下面实现的函数 **imageBoundingRect()** 用于从 canvas 中计算出绘制的图像的矩形范围，然后可以调用 **getImageData(x, y, w, h)** 从 cavans 中得到此范围的子图像。

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>裁剪图片</title>
    <style>
        canvas {
            border: 1px solid #CCC;
            border-radius: 4px;
            margin: 20px;
        }
    </style>
</head>

<body>
    <canvas id="canvas" class="canvas" width="400" height="300"></canvas>

    <script src="http://cdn.bootcss.com/jquery/1.9.1/jquery.min.js"></script>
    <script>
        var canvas  = $('#canvas').get(0);
        var context = canvas.getContext('2d');

        // 清楚 canvase 背景，使用的颜色是透明色 rgba(0, 0, 0, 0)
        context.clearRect(0, 0, canvas.width, canvas.height);

        // 画几个图形
        context.beginPath();
        context.fillStyle = '#CCC';
        context.rect(80, 30, 150, 50);
        context.rect(110, 110, 50, 50);
        context.moveTo(210, 10);
        context.lineTo(50, 50);
        context.lineTo(150, 100);
        context.fill();
        context.closePath();
    </script>

    <script>
        // 绘制的图形所在的矩形范围
        var bound = imageBoundingRect(canvas);

        // 获取 canvas 上 bound 内的 subimage，并把 subimage 绘制到新创建的 canvas 上
        var subimage = context.getImageData(bound.x, bound.y, bound.w, bound.h);
        var newCanvas = $('<canvas id="temp-canvas" width="' + bound.w + '" height="' + bound.h + '"></canvas>').get(0);
        var newContext = newCanvas.getContext('2d');
        newContext.putImageData(subimage, 0, 0);
        $(newCanvas).appendTo('body');

        /**
         * 去掉 canvas 中多余的边框，计算绘制的图形所在的矩形范围。
         *
         * @param  {Object}  canvas  画布 canvas
         * @param  {Integer} padding 边距
         * @return {Json}            返回 rect 对象，有 4 个属性：x, y, w, h
         */
        function imageBoundingRect(canvas, padding) {
            var imageData = context.getImageData(0, 0, canvas.width, canvas.height);
            var d = imageData.data; // d 是一个 rgba 的整数数组，每个像素在数组中占 4 个元素
            var w = imageData.width;
            var h = imageData.height;

            var bound = {
                minX: 100000,
                minY: 100000,
                maxX: 0,
                maxY: 0
            };

            for (var x = 0; x < w; x++) {
                for (var y = 0; y < h; y++) {
                    // 颜色的 r, g, b, a 分量
                    var r = d[y * w * 4 + x * 4 + 0];
                    var g = d[y * w * 4 + x * 4 + 1];
                    var b = d[y * w * 4 + x * 4 + 2];
                    var a = d[y * w * 4 + x * 4 + 3];

                    // 不是透明背景色的像素位置才参与计算
                    if (r !== 0 && g !== 0 && b !== 0 && a !== 0) {
                        bound.minX = Math.min(bound.minX, x);
                        bound.maxX = Math.max(bound.maxX, x);
                        bound.minY = Math.min(bound.minY, y);
                        bound.maxY = Math.max(bound.maxY, y);
                    }
                }
            }

            // 加上 padding
            var p = padding || 5;
            bound.minX = Math.max(bound.minX - p, 0);
            bound.minY = Math.max(bound.minY - p, 0);
            bound.maxX = Math.min(bound.maxX + p, w);
            bound.maxY = Math.min(bound.maxY + p, h);

            // 返回巨型的 Json 对象
            return {
                x: bound.minX,
                y: bound.minY,
                w: bound.maxX - bound.minX,
                h: bound.maxY - bound.minY
            };
        }
    </script>

</body>

</html>
```

