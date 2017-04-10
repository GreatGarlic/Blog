---
title: CSS3 缩放图片
date: 2017-04-10 13:26:26
tags: FE
---

CSS3 使用 `transform: scale(factor)` 就能够缩放图片，但是需要注意的是，**图片的 parent 需要设置 position 和 index 才有效**，否则会有 Bug:

![](/img/fe/scale-image-2.png)![](/img/fe/scale-image-1.png)<!--more-->

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>Scale Image</title>
    <style>
        body {
            padding: 100px;
        }

        .image-container {
            /* 最关键是 position 和 z-index */
            position: relative;
            z-index: 1;

            display: block;
            overflow: hidden;
            width: 200px;
            height: 200px;
            transition: all 0.3s;
            border: 4px solid grey;
            border-radius: 50%;
        }

        .image {
            width: 100%;
            height: 100%;
            transition: all 0.3s;
            opacity: 0.7;
        }

        .image-container:hover {
            border-color: #666;
            box-shadow: 0 0 4px grey;
        }

        .image-container:hover .image {
            transform: scale(1.4);
            opacity: 1;
        }
    </style>
</head>

<body>
    <a class="image-container" href="javascript:void(0)">
        <img class="image" src="http://omqpd0pt4.bkt.clouddn.com/ade.jpg">
    </a>

</body>

</html>
```

