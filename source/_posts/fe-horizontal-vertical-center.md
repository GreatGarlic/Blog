---
title: Div 在另一个 Div 中同时水平垂直居中
date: 2016-11-22 11:48:38
tags: FE
---
下面的方法可以使一个 Div 在另一个 Div 同时中水平垂直居中，这个效果例如在对话框中按钮布局在最下面时很有用  
![](/img/fe/vertical-horizontal-center.png)

<!--more-->

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <style media="screen">
        .outter {
            position: relative; /* 很重要，也可以是 absolute */
            height: 150px;
            border: 2px solid pink;
            margin: 50px auto;
        }
        .inner {
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            width: 80%;
            height: 25px; /* 很重要，需要有高度 */
            margin: auto; /* 很重要 */
            text-align: center; /* 内容水平居中，这里即按钮水平居中 */
            border: 2px dashed gray;
        }
        button {
            margin: 4px 20px;
            width: 80px;
        }
    </style>
</head>
<body>
    <div class="outter">
        <div class="inner">
            <button type="button" name="button">OK</button>
            <button type="button" name="button">Cancel</button>
        </div>
    </div>
</body>
</html>
```
