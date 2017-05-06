---
title: 拖拽普通 Element 到 zTree 
date: 2017-05-06 21:25:02
tags: FE
---

zTree 不支持 jQuery ui 的拖拽操作，为了拖拽普通的 element 到 zTree 上，需要自己实现拖拽功能，[Drag With Other DOMs](http://www.treejs.cn/v3/demo.php#_511) 演示了具体的实现，但是代码太多，不易于理解，这里把拖拽相关的核心代码提取出来，就能快速的理解拖拽的实现。<!--more-->

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title></title>
    <link rel="stylesheet" href="http://cdn.staticfile.org/zTree.v3/3.5.28/css/zTreeStyle/zTreeStyle.min.css">

    <style>
        #items {
            border: 1px solid #CCC;
            width: 400px;
            margin-top: 10px;
        }

        #items .item, .dragged-item {
            display: inline-block;
            width: 50px;
            height: 50px;
            line-height: 50px;
            text-align: center;
            cursor: pointer;
            border: 1px dashed orange;
            margin: 5px;
            background: rgba(255, 255, 0, 0.5);
        }
    </style>
</head>

<body>
    <ul id="treeDemo" class="ztree"></ul>

    <div id="items">
        <div class="item">1</div>
        <div class="item">2</div>
        <div class="item">3</div>
        <div class="item">4</div>
    </div>

    <script src="http://cdn.bootcss.com/jquery/3.2.1/jquery.min.js"></script>
    <script src="http://cdn.staticfile.org/zTree.v3/3.5.28/js/jquery.ztree.all.min.js"></script>
    <script src="http://cdn.staticfile.org/layer/2.3/layer.js"></script>

    <script>
        // DnD(Drag and Drop) 是拖拽实现的核心
        DnD = {
            $draggedElement: null, // 正在拖拽的 element
            // 按下鼠标
            mouseDown: function(e) {
                var $doc = $(document);
                var x = $doc.scrollLeft();
                var y = $doc.scrollTop();

                // 创建并保存拖拽的 element
                DnD.$draggedElement = $(e.target).clone();
                DnD.$draggedElement.addClass('dragged-item');
                DnD.$draggedElement.css({
                    'position': 'absolute',
                    'left': (e.clientX + x + 2) + 'px',
                    'top' : (e.clientY + y + 2) + 'px'
                });
                DnD.$draggedElement.appendTo('body');

                // 绑定鼠标移动，松开和选择文本事件
                $doc.on('mousemove',   DnD.mouseMove);
                $doc.on('mouseup',     DnD.mouseUp);
                $doc.on('selectstart', DnD.noSelect);
            },
            // 移动鼠标
            mouseMove: function(e) {
                // 移动拖拽的 element
                var $doc = $(document);
                var x = $doc.scrollLeft();
                var y = $doc.scrollTop();

                DnD.$draggedElement.css({
                    'left': (e.clientX + x + 3) + 'px',
                    'top' : (e.clientY + y + 3) + 'px'
                });

                return false;
            },
            // 松开鼠标
            mouseUp: function(e) {
                // 松开鼠标时取消 document 绑定的事件
                var $doc = $(document);
                $doc.off('mousemove',   DnD.mouseMove);
                $doc.off('mouseup',     DnD.mouseUp);
                $doc.off('selectstart', DnD.noSelect);

                // 删除拖拽的 element
                DnD.$draggedElement.remove();
                DnD.$draggedElement = null;
            },
            // 取消选择，为了实现效果好一些，所以在拖拽时不允许选择文本
            noSelect: function() {
                return false;
            }
        };

        $(document).on('mousedown', '.item', DnD.mouseDown);
    </script>

    <script>
        var setting = {
            data: {
                simpleData: {
                    enable : true, // 启用数组结构的数据创建 zTree
                    idKey  : 'id', // 定义 node id 的 key，可自定义
                    pIdKey : 'parentId', // 定义 parent id 的 key，可自定义
                    rootPId: 0 // 根节点的 parentId，这个节点是看不到的，因为不存在
                }
            },
            callback: {
                onMouseUp: function(event, treeId, treeNode) {
                    // 拖放到节点上
                    if (treeNode && DnD.$draggedElement) {
                        layer.msg(DnD.$draggedElement.text() + '-' + treeNode.name);
                    }
                }
            }
        };
        var nodes = [
            { 'id': 1, 'parentId': 0, 'name': '试卷', open: true },
            { 'id': 2, 'parentId': 1, 'name': '数学' },
            { 'id': 3, 'parentId': 1, 'name': '历史' },
            { 'id': 4, 'parentId': 2, 'name': '参数方程' },
            { 'id': 5, 'parentId': 2, 'name': '贝塞尔曲线' },
            { 'id': 6, 'parentId': 3, 'name': '玄武门事变' },
            { 'id': 7, 'parentId': 3, 'name': '桃园三结义' }
        ];

        var tree = $.fn.zTree.init($('#treeDemo'), setting, nodes);
    </script>
</body>

</html>
```



