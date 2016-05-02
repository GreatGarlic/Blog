---
title: jQuery ui 拖拽
date: 2016-05-02 14:06:11
tags: FE
---

`jQuery ui` 提供了拖拽元素，拖拽排序等功能，例如要让一个元素能够被拖拽，只要给它执行 `draggable()` 函数即可，要实现 `ul li` 拖拽排序，只要在 ui 上执行函数 `sortable()` 就能实现了，具体更多的例子请参考 [jQuery UI 实例 - 拖动](http://www.runoob.com/jqueryui/example-draggable.html)

<!--more-->

## 从一个容器拖动到另一个容器
如图，从上面把色块拖动到下面，核心代码就 2 个函数的调用 `draggable()` 和 `droppable()`  
![](/img/fe/drag-and-drop.png)

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <style>
    #src, #dst {
        border: 1px solid gray;
        width: 400px;
        height: 100px;
        margin-top: 10px;
    }
    .item {
        display: inline-block;
        float: left;
        width: 50px;
        height: 50px;
        background-color: red;
        cursor: pointer;
        line-height: 50px;
        text-align: center;
    }

    .item-1 {
        background: red;
    }

    .item-2 {
        background: green;
    }

    .item-3 {
        background: blue;
    }

    .item-4 {
        background: yellow;
    }

    .ui-draggable-dragging {
        opacity: 0.5;
    }
    </style>
</head>
<body>
    <div id="src">
        <div class="item item-1">1</div>
        <div class="item item-2">2</div>
        <div class="item item-3">3</div>
        <div class="item item-4">4</div>
    </div>
    <div id="dst"></div>

    <script src="../jquery.min.js"></script>
    <script src="../jquery-ui.min.js"></script>
    <script>
        $(document).ready(function() {
            $('.item').draggable({
                helper : 'clone',
                cursor: 'move',
                revert: 'invalid' // 拖拽失败时漂回原来的位置
            });
        });

        $('#dst').droppable({
            drop: function(event, ui) {
                $('#dst').append(ui.draggable);
            }
        });
    </script>
</body>
</html>
```

## 列表拖动排序
如图，拖动左边的 list，同时右边对应的信息也进行排序，核心代码就 1 个函数的调用 `sortable()`  
![](/img/fe/list-sort.png)

```html

<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>jQuery 对元素拖动排序</title>
    <style type="text/css">
        #animals {
            cursor: pointer;
            width: 200px;
        }

        li {
            /*float:left; */
            display: block;  /*需要是 block 或者 float 的*/
            list-style-type: none;
            margin-top: 10px;
        }

        #animals li:hover {
            background: #EEE;
        }

        #left {
            float: left;
        }

        #right {
            float: left;
        }

        #info li {
            box-shadow: 0 0 2px #DDD;
        }
    </style>
</head>
<body>
    <div id="content-wrap">
        <div id="left">
            <ul id="animals">
                <li data-order="1">(☆_☆)</li>
                <li data-order="2">凸^-^凸</li>
                <li data-order="3">(^_−)−☆</li>
                <li data-order="4">(◐‿◑)﻿
                    <ul id="inner">
                        <li data-order="21">(☆_☆)</li>
                        <li data-order="22">凸^-^凸</li>
                        <li data-order="23">(^_−)−☆</li>
                        <li data-order="24">☆*:.｡. o(≧▽≦)o .｡.:*☆</li>
                    </ul>
                </li>
            </ul>
        </div>

        <div id="right">
            <ul id="info">
                <li data-order="1">(☆_☆)</li>
                <li data-order="2">凸^-^凸</li>
                <li data-order="3">(^_−)−☆</li>
                <li data-order="4">(◐‿◑)﻿</li>
            </ul>
        </div>
    </div>

    <script type="text/javascript" src="../jquery.min.js"></script>
	<script type="text/javascript" src="../jquery-ui.min.js"></script>
    <script type="text/javascript">
        $(document).ready(function () {
            // 启用拖拽排列操作
            $("#animals").sortable({
                opacity: 0.5,
                update: function (event, ui) { // 拖拽完成后的回调函数，例如保存到数据库等
                    var order = ui.item.attr('data-order'); // 被拖拽的元素: ui.item
                    var preOrder = ui.item.prev().attr('data-order');

                    if (preOrder) {
                        $('#info li[data-order="' + order + '"]').insertAfter($('#info li[data-order="' + preOrder + '"]'));
                    } else {
                        $('#info').prepend($('#info li[data-order="' + order + '"]'));
                    }

                    regenerateOrder();
                }
            });
            $("#inner").sortable();
        });

        // 重新生成 order
        function regenerateOrder() {
            var order = 1;
            $('#animals > li').each(function() {
                $(this).attr('data-order', order);
                order++;
            });

            order = 1;
            $('#info > li').each(function() {
                $(this).attr('data-order', order);
                order++;
            });
        }
    </script>
</body>
</html>
```
