---
title: 事件的坐标
date: 2017-08-19 15:10:09
tags: FE
---

JS 的事件有几个重要的坐标:

* **(offsetX, offsetY)**: 事件触发点在事件源元素的坐标系统中的坐标(相对于元素的左上角，左上角坐标为 (0, 0))
* **(pageX, pageY)**: 事件触发点相对于整个页面左上角的坐标，包括了滚动条隐藏的部分
* **(clientX, clientY)**: 事件触发点相对于页面可视部分(客户区)左上角的坐标，不包括滚动条隐藏的部分
* **(screenX, screenY)**: 事件触发点相对于屏幕左上角的坐标

<!--more-->

使用下面的代码来验证以上坐标的说法:

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title></title>
    <script src="http://cdn.bootcss.com/jquery/1.9.1/jquery.min.js"></script>
    <style>
        html, body {
            margin: 0;
            padding: 0;
        }

        div {
            margin-top: 1100px;
            margin-left: 1100px;
            width: 400px;
            height: 400px;
            background: grey;
        }
    </style>
</head>

<body>
    <div></div>

    <script>
        $('div').click(function(e) {
            console.log('---------------------------');
            console.log(`offset: (${e.offsetX}, ${e.offsetY})`);
            console.log(`client: (${e.clientX}, ${e.clientY})`);
            console.log(`screen: (${e.screenX}, ${e.screenY})`);
            console.log(`page:   (${e.pageX}, ${e.pageY})`);
        });
    </script>
</body>

</html>
```

点击 div 上坐标为 (12, 7) 处，输出:

```
offset: (12, 7)
client: (331, 424)
screen: (1051, 454)
page:   (1112, 1107)
```

由于 html 和 body 的 margin 和 padding 都为 0，所以 div 的左上角的坐标为 (1100, 1100)。因为 offset 和 page 的坐标与浏览器的大小，在屏幕的位置无关，所以能够直接计算出来，offset 为 (12, 7)，page 为 (1112, 1107) 都是正确的。Client 和 screen 的坐标与浏览器的位置和大小有关，需要根据实际情况进行计算，当然上面输出的结果是对的。