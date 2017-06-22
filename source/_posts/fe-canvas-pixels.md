---
title: Canvas 像素数据处理
date: 2017-06-22 15:18:52
tags: FE
---

Canvas 的 context 调用 getImageData() 获取 canvas 中图片的像素数据，处理好后再调用 putImageData() 设置回 canvas。

```js
var canvas = $('#canvas').get(0);
canvas.width = 500; // canvas 的实际宽度，默认是 300
canvas.height = 300;
var ctx = canvas.getContext('2d');

var imageData = ctx.getImageData(0, 0, canvas.width, canvas.height); // 获取像素数据
grayscale(imageData); // 处理像素数据
ctx.putImageData(imageData, 0, 0); // 设置回 canvas
```

<!--more-->

## 灰度效果

```js
// 灰度效果（grayscale）就是取红、绿、蓝三个像素值的算术平均值，这实际上将图像转成了黑白形式。
function grayscale(pixels) {
    var d = pixels.data; // d 是一个 rgba 的整数数组

    for (var i = 0; i < d.length; i += 4) {
        var r = d[i];
        var g = d[i + 1];
        var b = d[i + 2];
        d[i] = d[i + 1] = d[i + 2] = (r + g + b) / 3;
    }
}
```

> d 是一个像素颜色分量的整数数组，每 4 个值对应一个像素的 4 个颜色分量 rgba:
>
> * d[i+0] 为红色值
> * d[i+1] 为绿色值
> * d[i+2] 为蓝色值
> * d[i+3] 为 alpha 通道值

## 复古效果

```js
// 复古效果（sepia）则是将红、绿、蓝三个像素，分别取这三个值的某种加权平均值，使得图像有一种古旧的效果。
function siepa(pixels) {
    var d = pixels.data; // d 是一个 rgba 的整数数组

    for (var i = 0; i < d.length; i += 4) {
        var r = d[i];
        var g = d[i + 1];
        var b = d[i + 2];

        d[i] = (r * 0.393) + (g * 0.769) + (b * 0.189); // red
        d[i + 1] = (r * 0.349) + (g * 0.686) + (b * 0.168); // green
        d[i + 2] = (r * 0.272) + (g * 0.534) + (b * 0.131); // blue
    }
}
```

## 红色蒙版

```js
// 红色蒙版指的是，让图像呈现一种偏红的效果。
// 算法是将红色通道设为红、绿、蓝三个值的平均值，而将绿色通道和蓝色通道都设为 0。
function red(pixels) {
    var d = pixels.data; // d 是一个 rgba 的整数数组

    for (var i = 0; i < d.length; i += 4) {
        var r = d[i];
        var g = d[i + 1];
        var b = d[i + 2];

        d[i] = (r + g + b) / 3; // 红色通道取平均值
        d[i + 1] = d[i + 2] = 0; // 绿色通道和蓝色通道都设为 0
    }
}
```

## 亮度效果

```js
// 亮度效果（brightness）是指让图像变得更亮或更暗。算法将红色通道、绿色通道、蓝色通道，同时加上一个正值或负值。
function brightness(pixels, delta) {
    var d = pixels.data; // d 是一个 rgba 的整数数组

    for (var i = 0; i < d.length; i += 4) {
        d[i] += delta;
        d[i + 1] += delta;
        d[i + 2] += delta;
    }
}
```

## 反转效果

```js
// 反转效果（invert）是指图片呈现一种色彩颠倒的效果。算法为红、绿、蓝通道都取各自的相反值（255-原值）。
function invert(pixels, delta) {
    var d = pixels.data; // d 是一个 rgba 的整数数组

    for (var i = 0; i < d.length; i += 4) {
        d[i] = 255 - d[i];
        d[i + 1] = 255 - d[i + 1];
        d[i + 2] = 255 - d[i + 2];
    }
}
```

