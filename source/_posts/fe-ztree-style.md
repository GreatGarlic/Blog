---
title: zTree 修改 Awesome Style
date: 2017-04-15 17:21:40
tags: [FE, zTree]
---

zTree 默认的皮肤很不好看，还是 XP 的风格，例子中的 Awesome Style 很不错：

![](/img/fe/ztree-1.png)

上面红色的 Awesome Style 风格并不适合所有情况，这里就对它进行简单的修改，下面是效果图：

![](/img/fe/ztree-2.png)

<!--more-->

* 需要引入 font-awesome.css 和 awesome.css，并且不要引入 zTreeStyle.css
* 注册函数 addHoverDom 和 removeHoverDom
* 覆盖 Awesome Style 需要修改的 CSS

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>zTree</title>

    <!-- [1] 需要引入 font-awesome.css 和 awesome.css，并且不要引入 zTreeStyle.css -->
    <link rel="stylesheet" href="http://cdn.staticfile.org/font-awesome/4.7.0/css/font-awesome.min.css">
    <link rel="stylesheet" href="http://cdn.staticfile.org/zTree.v3/3.5.28/css/awesomeStyle/awesome.min.css">
    <style media="screen">
        .node_name {
            font-size: 14px;
            margin-left: 2px;
        }

        .ztree li a {
            margin-bottom: 3px;
        }

        .ztree, .ztree * {
            background: white;
        }

        .ztree, .ztree li a,
        .ztree li span.button::before,
        .ztree li span.button.ico_open::before,
        .ztree li span.button.ico_close::before,
        .ztree li span.button.ico_docu::before,
        span.tmpzTreeMove_arrow::before {
            color: #444;
        }
        .ztree li a.curSelectedNode {
            background-color: white;
            color: darkred;
        }
    </style>
</head>

<body>
    <ul id="treeDemo" class="ztree"></ul>

    <script src="http://cdn.bootcss.com/jquery/3.2.1/jquery.min.js"></script>
    <script src="http://cdn.staticfile.org/zTree.v3/3.5.28/js/jquery.ztree.all.min.js"></script>
    <script>
        var setting = {
            data: {
                simpleData: {
                    enable:  true,       // 启用数组结构的数据创建 zTree
                    idKey:   'id',       // 定义 node id 的 key，可自定义
                    pIdKey:  'parentId', // 定义 parent id 的 key，可自定义
                    rootPId: 0           // 根节点的 parentId，这个节点是看不到的，因为不存在
                }
            },
            edit: {
                enable: true // [*] 启用拖拽，重命名和删除等编辑操作
            },
            view: {
                selectedMulti: false,     // [*] 为 true 时可选择多个节点，为 false 只能选择一个，默认为 true
                addHoverDom: addHoverDom, // [2] 注册函数 addHoverDom 和 removeHoverDom
                removeHoverDom: removeHoverDom
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

        var newCount = 10000000;

        // [2.1] 鼠标移动到节点上时创建一个新建子节点的按钮
        function addHoverDom(treeId, treeNode) {
            if (treeNode.editNameFlag || $("#addBtn_"+treeNode.tId).length>0) return;

            var $nodeName = $("#"+treeNode.tId+"_span");
            $nodeName.after('<span class="button add" id="addBtn_'+treeNode.tId+'" title="新建子节点" onfocus="this.blur();"></span>');

            var btn = $("#addBtn_"+treeNode.tId);
            if (btn) {
                btn.bind("click", function() {
                    tree.addNodes(treeNode, 0, {id: newCount++, parentId: treeNode.id, name: "请重命名..."});
                    return false;
                });
            }
        };

        // [2.2] 鼠标从节点上离开时删除新建子节点的按钮
        function removeHoverDom(treeId, treeNode) {
            $("#addBtn_"+treeNode.tId).unbind().remove();
        };
    </script>
</body>

</html>
```

