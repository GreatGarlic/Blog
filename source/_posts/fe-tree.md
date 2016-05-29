---
title: HTML 树的实现
date: 2016-05-29 10:18:16
tags: FE
---
树形结构的使用很广泛，例如用来显示文件夹和文件，组织机构的表示等，可以使用 `zTree` 来实现，这里我们将自己实现树，了解其原理。

实现下图中表示文件的树，主要是使用 `<ul>` 和 `<li>` 组织结构、`CSS` 调整显示效果、`jQuery` 实现点击的动态效果和 `jQuery UI` 实现拖拽操作 (主要的代码都在 HTML 和 CSS 上，JS 的代码只有 20 行)。

![](/img/fe/tree/result.png)

<!--more-->

## 关键技术点
* 目录就是文件夹，文件夹是特殊的文件
* 文件夹的内容放在 `<ul>` 中
* 文件夹和文件使用 `<li>` 表示
* 文件夹和文件的名字使用 `<li>` 中的 `<div class="file">` 显示，文件夹有类 `is-directory`，文件的话使用类 `is-file`

    ```html
    <li> <!-- 目录 -->
        <div class="file is-directory"> <!-- 目录的名字, 图标 -->
            <span class="branch"></span> <!-- 目录的箭头 -->
            <span class="icon"></span> <!-- 目录或文件的图标 -->
            <span class="name">学科</span> <!-- 目录或文件的名字 -->
        </div>

        <ul> <!-- 目录下的文件或子目录 -->
            <li> <!-- 文件 -->
                <div class="file is-file">
                    <span class="branch"></span>
                    <span class="icon"></span>
                    <span class="name">百科目录</span>
                </div>
            </li>
    ```
* 文件夹展开时对应的 `<div>` 加上 CSS 类 `expand` 同时移除类 `collapse`，收缩时则相反
* 文件被点击时对应的 `<div>` 加上 CSS 类 `active` 同时移除其他 `<div>` 上的类 `active`
    
## 实现代码
```html
<!DOCTYPE html>
<html>
<head>
    <title>Directory Tree</title>
    <meta charset="UTF-8">

    <style>
    body {
        font-family: "微软雅黑", "HelveticaNeue-Light", "Helvetica Neue Light", "Helvetica Neue", Helvetica, Arial, sans-serif;
    }

    #file-tree ul {
        list-style: none;
        padding-left: 20px;
    }

    #file-tree .file,
    #file-tree .branch,
    #file-tree .icon,
    #file-tree .name {
        height: 20px;
        line-height: 20px;
        display: inline-block;
        vertical-align: middle;
    }

    #file-tree .file {
        display: inline-block;
        font-size: 0;
        border: 1px solid transparent;
    }

    #file-tree .file.active {
        border-radius: 2px;
        border: 1px solid rgb(68, 211, 255);
    }

    #file-tree .is-directory {
        margin-left: -15px;
    }

    /* Bruanch */
    #file-tree .is-directory .branch {
        width: 15px;

    }

    #file-tree .is-file .branch {
        width: 0;

    }

    #file-tree .is-directory .branch {
        cursor: pointer;
    }

    #file-tree .is-directory.expand .branch {
        background: url(arrow.png) no-repeat -65px 3px;
    }

    #file-tree .is-directory.collapse .branch {
        background: url(arrow.png) no-repeat -32px 3px;
    }

    /* icon */
    #file-tree .icon {
        width: 20px;
    }

    #file-tree .is-directory .icon {
        background: url(folder.png) no-repeat;
    }

    #file-tree .is-file .icon {
        background: url(file.png) no-repeat;
    }

    /* name */
    #file-tree .name {
        cursor: default;
        font-size: 14px;
        padding: 0 3px;
    }

    #file-tree .file.active .name {
        background: rgb(68, 211, 255);
    }
    </style>
</head>
<body>
    <!-- 目录中的文件或者子目录放在 <ul> 里, 文件或目录是一个 <li>，文件或目录的名字是 <li> 中的 <div> -->
    <div id="file-tree">
        <ul> <!-- 根目录所在目录 -->
            <li> <!-- 根目录 -->
                <div class="file is-directory"> <!-- 根目录的名字, 图标 -->
                    <span class="branch"></span>
                    <span class="icon"></span>
                    <span class="name">根目录</span>
                </div>

                <ul> <!-- 根目录下的文件或子目录 -->
                    <li> <!-- 目录 -->
                        <div class="file is-directory"> <!-- 目录的名字, 图标 -->
                            <span class="branch"></span>
                            <span class="icon"></span>
                            <span class="name">学科</span>
                        </div>

                        <ul> <!-- 目录下的文件或子目录 -->
                            <li> <!-- 文件 -->
                                <div class="file is-file">
                                    <span class="branch"></span>
                                    <span class="icon"></span>
                                    <span class="name">百科目录</span>
                                </div>
                            </li>
                            <li> <!-- 文件 -->
                                <div class="file is-file">
                                    <span class="branch"></span>
                                    <span class="icon"></span>
                                    <span class="name">操作指南</span>
                                </div>
                            </li>
                            <li> <!-- 文件 -->
                                <div class="file is-file">
                                    <span class="branch"></span>
                                    <span class="icon"></span>
                                    <span class="name">README.md</span>
                                </div>
                            </li>
                        </ul>
                    </li>
                    <li> <!-- 文件 -->
                        <div class="file is-file"> <!-- 文件的名字, 图标 -->
                            <span class="branch"></span>
                            <span class="icon"></span>
                            <span class="name">README.md</span>
                        </div>
                    </li>
                    <li> <!-- 文件 -->
                        <div class="file is-file"> <!-- 文件的名字, 图标 -->
                            <span class="branch"></span>
                            <span class="icon"></span>
                            <span class="name">操作指南</span>
                        </div>
                    </li>
                </ul>
            </li>
        </ul>
    </div>

    <script src="../jquery.min.js"></script>
    <script src="../jquery-ui.min.js"></script>
    <script>
    $(document).ready(function() {
        $('.is-directory').addClass('expand'); // 开始时都是展开的

        // 点击箭头展开或者收缩
        $('#file-tree').on('click', '.is-directory .branch', function() {
            var $directory = $(this).parent();
            $directory.toggleClass('expand').toggleClass('collapse');
            $directory.siblings('ul').slideToggle('fast');
        });

        // 高亮点击的文件
        $('#file-tree').on('mousedown', '.name', function(e) {
            $('.name').parent().removeClass('active');
            $(this).parent().addClass('active');
        });

        // 文件可以拖动
        $('.is-file').draggable({
            helper : 'clone',
            cursor: 'move',
            revert: 'invalid'
        });
    });
    </script>
</body>
</html>
```

## 资源文件
![](/img/fe/tree/arrow.png)
![](/img/fe/tree/file.png)
![](/img/fe/tree/folder.png)
