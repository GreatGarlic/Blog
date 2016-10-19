---
title: Lean Modal
date: 2016-10-19 14:45:42
tags: FE
---
LeanModal 是一个用于创建模式对话框的超级简单 jQuery插件。可以展示隐藏的页面内容，整个插件大小只有不到 1K，可灵活变化高度和宽度，没有用到任何图片，支持在一个页面中创建多个实例，非常适合于创建：登录框，注册框，警告对话框等。

<!--more-->

## 使用步骤
1. 将 jQuery，jquery.leanModal.min.js 添加页面

    ```html
    <script src="jquery.js" charset="utf-8"></script>
    <script src="jquery.leanModal.min.js" charset="utf-8"></script>
    ```
2. 定义对话框的 Element

    ```html
    <div id="dialog-1">
        <p>LeanModal是一个用于创建模式对话框的超级简单JQuery插件。可以展示隐藏的页面内容，整个插件大小只有780bytes，
            可灵活变化高度和宽度，没有用到任何图片，支持在一个页面中创建多个实例，非常适合于创建：登录框，
            注册框，警告对话框等。最重要的是界面非常干净UI设计很不错。
        </p>
    </div>
    ```
3. 定义点击弹出对话框的触发器，例如 `<a>` 或者 `<button>`

    ```html
    <a id="modal-trigger" href="#dialog-1">显示对话框</a>
    ```

    > LeanModal 的触发器必须有 `href` 属性，其值为对话框的 `id`
4. 初始化 LeanModal

    ```js
    $('#modal-trigger').leanModal();
    ```
5. 如果想要一个 Mask 层，定义 `<div id="lean_overlay"></div>` 并使用 `CSS` 给它添加样式(id 必须是 `lean_overlay`)

6. 关闭有 Mask 层的 Lean Modal

    ```js
    $("#lean_overlay").click();
    ```

## 完整例子
```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>Lean Modal</title>
    <script src="jquery.js" charset="utf-8"></script>
    <script src="jquery.leanModal.min.js" charset="utf-8"></script>
    <style media="screen">
        #dialog-1, #dialog-2 {
            z-index:1000;
            display: none;
            background: white;
            border: solid 1px gray;
            border-radius: 8px;
            box-shadow: 2px 2px 8px #444;
            padding: 5px 10px;
            width: 400px;
            height: 200px;
            overflow: scroll;
        }

        #lean_overlay {
            position: fixed;
            z-index: 100;
            top: 0px;
            left: 0px;
            height:100%;
            width:100%;
            background: #000;
            opacity: 0.4;
            display: none;
        }
    </style>
</head>

<body>
    <!-- 显示对话框的链接 -->
    <a href="#dialog-1">显示对话框</a>
    <!-- 显示对话框的按钮 -->
    <button href="#dialog-2">显示对话框</button>

    <!-- 对话框 -->
    <div id="dialog-1">
        <p>LeanModal是一个用于创建模式对话框的超级简单JQuery插件。可以展示隐藏的页面内容，整个插件大小只有780bytes，
            可灵活变化高度和宽度，没有用到任何图片，支持在一个页面中创建多个实例，非常适合于创建：登录框，
            注册框，警告对话框等。最重要的是界面非常干净UI设计很不错。
        </p>
    </div>

    <!-- 对话框 -->
    <div id="dialog-2">
        <p>You can define new user types in QStandardItem subclasses to ensure that custom items are treated specially; for example, when items are sorted.
        </p>
    </div>
    
    <!-- LeanModal 对话框的 Mask -->
    <div id="lean_overlay"></div>

    <script>
        $(document).ready(function() {
            $('a, button').leanModal();
        });
    </script>
</body>

</html>
```

## 参考资料
* [LeanModal 轻量级jquery弹出层插件](http://www.jqueryfuns.com/resource/44)
* [Bare bones modal dialog windows](http://leanmodal.finelysliced.com.au/#)
