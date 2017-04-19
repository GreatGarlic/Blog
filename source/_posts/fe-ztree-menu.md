---
title: zTree 右键菜单
date: 2017-04-20 07:11:15
tags: FE
---

zTree 使用右边的添加、编辑、删除按钮时容易误操作（踩过坑了），所以下面使用右键菜单来编辑 zTree。

既然需要对树进行编辑，说明数据是需要保存到服务器的，所以树的加载也是需要使用动态加载，不过事例中写死是为了无服务器时便与演示（注释中有动态访问服务器的代码）。

**实现时需要注意几点:**

* 重命名操作需要在 zTree 的回调函数 beforeRename 中进行
* 创建新节点时
  * 节点没有展开，先展开它，这时如果还没有从服务器加载过数据，则会先从服务器中加载子节点数据，展开完成后会调用回调函数 onExpand，这时在 onExpand 中创建新节点。节点没有展开，且没有从服务器加载过子节点数据时直接创建新节点，就会很有可能看到 2 个一样新建的节点，因为调用 window.tree.addNodes 创建了一个节点，服务器返回的数据中很可能也已经有这个节点的数据，所以重复了，不过在数据库中并没有重复，只是浏览器本地的 zTree 中内存数据重复
  * 是根节点，或者非根节点且已经展开了，则直接创建新节点，这时不会从服务器加载子节点数据

**使用时需要修改 TODO 相关的部分:**

* 配置 setting.async 从服务器加载数据，并删除 nodes 初始化代码

* 完成函数 createNode(), removeNode(), renameNode()，已有相关的注释作为提示

  > $.rest.syncCreate(), $.rest.syncRemove(), $.rest.syncUpdate() 是对 $.ajax() 的简单封装，为了简化 Ajax 请求操作，可以自己实现或则参考 [jQuery 的 REST 插件](http://qtdebug.com/fe-rest/)

<!--more-->

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>zTree</title>
    <link rel="stylesheet" href="http://cdn.staticfile.org/zTree.v3/3.5.28/css/zTreeStyle/zTreeStyle.min.css">

    <style media="screen">
        body {
            font-family: "微软雅黑", "HelveticaNeue-Light", "Helvetica Neue Light", "Helvetica Neue", Helvetica, Arial, sans-serif;
            font-size: 14px;
        }

        /* 菜单的样式 */
        #directory-tree-menu {
            position:absolute;
            top:0;
            display: none;
            margin: 0;
            padding: 6px 0;
            text-align: left;
            border: 1px solid #BFBFBF;
            border-radius: 3px;
            background-color: #EEE;
            box-shadow: 0 0 10px #AAA;
        }

        #directory-tree-menu li {
            padding: 2px 15px;
            list-style: none outside none;
            cursor: default;
            color: #666;
        }
        #directory-tree-menu li:hover {
            color: #EEE;
            background-color: #666;
        }

        li#menu-item-delete, li#menu-item-rename {
            margin-top: 1px;
        }
    </style>
</head>
<body>
    <ul id="directory-tree" class="ztree"></ul>

    <!-- 目录树的右键菜单 -->
	<ul id="directory-tree-menu">
		<li id="menu-item-create">新建文件夹</li>
		<li id="menu-item-delete">删除文件夹</li>
		<li id="menu-item-rename">重命名</li>
	</ul>

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
                enable: true,
                showRemoveBtn: false,
                showRenameBtn: false
            },
            view: {
                selectedMulti: false, // [*] 为 true 时可选择多个节点，为 false 只能选择一个，默认为 true
            },
            // TODO: 配置从服务器 AJAX 加载节点数据
            // async: {
            //     enable: true,
            //     type: 'get',
            //     url: function(treeId, treeNode) {
            //         // treeNode 不存在时表示需要加载根节点下的数据
            //         return Urls.REST_DIRECTORIES + '?parentDirectoryId=' + (treeNode ? treeNode.directoryId : 0);
            //     }
            // },
            callback: {
                beforeRename: renameNode,
                onRightClick: showMenu,
                onExpand: function(event, treeId, treeNode) {
                    // 展开后再创建
                    if (window.needCreateNewNode) {
                        window.needCreateNewNode = false;
                        createNode(window.newNodeParentNode);
                    }
                }
            }
        };

        // TODO: 实际使用时删除下面的 nodes
        var nodes = [
            { 'id': 1, 'parentId': 0, 'name': '试卷', open: true },
            { 'id': 2, 'parentId': 1, 'name': '数学' },
            { 'id': 3, 'parentId': 1, 'name': '历史' },
            { 'id': 4, 'parentId': 2, 'name': '参数方程' },
            { 'id': 5, 'parentId': 2, 'name': '贝塞尔曲线' },
            { 'id': 6, 'parentId': 3, 'name': '玄武门事变' },
            { 'id': 7, 'parentId': 3, 'name': '桃园三结义' }
        ];

        // 为了简单演示，节点数据写死，实际中应该是从服务器动态加载，
        // 毕竟写死的数据就没必要动态创建，删除和重命名了
        window.tree = $.fn.zTree.init($('#directory-tree'), setting, nodes);

        // 点击创建文件夹
        $('#menu-item-create').click(function(event) {
            hideMenu();
            var parentNode = window.tree.getSelectedNodes()[0];

            // 如果 parentNode 没有展开，则先展开 parentNode，展开时会先从服务器加载子节点数据，
            // 展开后在回调函数 onExpand 中创建新的 node
            // 如果 parentNode 不存在，或则存在且已经展开，则直接创建新的 node
            //
            // 如果不这么做，parentNode 未展开时创建新 node，会出现重复的新 node
            // createNode 时 window.tree.addNodes() 创建了一个到 tree 中，服务器返回的 node 中还包含了新创建的这个 node
            // 导致看到了 2 个同样新创建的 node，不过在数据库中还是只有一个 node，只是显示上重复
            if (parentNode && !parentNode.open) {
                window.needCreateNewNode = true;
                window.newNodeParentNode = parentNode;
                window.tree.expandNode(parentNode, true, false, true, true);
            } else {
                createNode(parentNode); // 根目录下创建文件夹
            }
        });

        // 点击删除文件夹
        $('#menu-item-delete').click(function(event) {
            hideMenu();
            var nodes = window.tree.getSelectedNodes();

            if (nodes.length === 0) { return; }
            if (!confirm('确定删除 ' +  nodes[0].name + ' 吗？')) { return; } // 删除前最好是询问一下是否确定删除，防止误操作

            removeNode(nodes[0]); // 只删除第一个
        });

        // 点击重命名
        $('#menu-item-rename').click(function(event) {
            hideMenu();
            var nodes = window.tree.getSelectedNodes();

            if (nodes.length === 0) { return; }

            window.tree.editName(nodes[0]);
        });

        // 显示菜单
        function showMenu(event, treeId, treeNode) {
            var type = '';
            var x = event.clientX;
            var y = event.clientY;

            if (!treeNode) {
                type = 'root';
                window.tree.cancelSelectedNode();
            } else if (treeNode && !treeNode.noR) { // noR 属性为 true 表示禁止右键菜单
                window.tree.selectNode(treeNode);
            }

            // 不同节点显示的菜单可能不一样
            if ('root' === type) {
                $('#menu-item-delete').hide();
                $('#menu-item-rename').hide();
            } else {
                $('#menu-item-delete').show();
                $('#menu-item-rename').show();
            }

            $('#directory-tree-menu').css({left: x+'px', top: y+'px'}).show();

            $(document).on('mousedown', function(event) {
                if (!(event.target.id == 'directory-tree-menu' || $(event.target).parents('#directory-tree-menu').length>0)) {
                    hideMenu();
                }
            });
        }

        function hideMenu() {
            $('#directory-tree-menu').hide();
            $(document).off('mousedown');
        }

        /**
         * 在 parentNode 下创建新节点
         *
         * @param  {object} parentNode 新创建子节点的父节点
         * @return 无返回值
         */
        function createNode(parentNode) {
            // TODO: 同步 AJAX 请求服务器创建 node 成功后添加 node 到 tree 中
            // var name = '新建文件夹';
            // var parentDirectoryId = parentTreeNode ? parentTreeNode.directoryId : 0;
            //
            // $.rest.syncCreate({url: Urls.REST_DIRECTORIES, data: {name: name, parentDirectoryId: parentDirectoryId}, success: function(result) {
            //     if (result.success) {
            //         var directoryId = result.data.directoryId;
            //         var node = {directoryId: directoryId, parentDirectoryId: parentDirectoryId, name: name, isParent: true};
            //         window.tree.addNodes(parentTreeNode, -1, node, false);
            //     } else {
            //         layer.msg(result.message);
            //     }
            // }});
        }

        /**
         * 删除节点
         * @param  {object} treeNode 需要删除的节点
         * @return 无返回值
         */
        function removeNode(treeNode) {
            // TODO: Ajax 同步请求服务器删除数据库中的节点，成功则从 tree 中删除此 node
            // $.rest.syncRemove({url: Urls.REST_DIRECTORIES_WITH_ID, urlParams: {directoryId: treeNode.directoryId}, success: function(result) {
            //     if (result.success) {
            //         window.tree.removeNode(treeNode);
            //     } else {
            //         layer.msg(result.message);
            //     }
            // }});
        }

        /**
         * 重命名节点
         * @param  {object}  treeId   [description]
         * @param  {object}  treeNode 需要重命名的节点
         * @param  {string}  newName  新名字
         * @param  {bool}    isCancel 按 ESC 取消重命名时为 true，否则为 false
         * @return 成功重命名返回 true，否则返回 false，isCancel 为 true 时时什么都不返回
         */
        function renameNode(treeId, treeNode, newName, isCancel) {
            // 1. 按下 ESC 取消重命名时 isCancel 为 true，否则为 false
            // 2. 把名字前后的空格去掉
            // 3. 名字不能为空或则全是空格
            // 4. 请求服务器重命名
            //    4.1. 成功则返回 true 更新目录树上目录的名字
            //    4.2. 失败返回 false 并提示错误信息

            if (!isCancel) {
                newName = $.trim(newName);

                if (!newName) {
                    alert('目录名不能为空');
                    return false;
                }

                // TODO: 同步 AJAX 请求服务器重命名 node，成功后返回 true
                var ok = false;
                // $.rest.syncUpdate({url: Urls.REST_DIRECTORY_NAME, urlParams: {directoryId: treeNode.directoryId},
                //     data: {name: newName}, success: function(result) {
                //     ok = result.success;
                //
                //     if (!ok) {
                //         layer.msg(result.message);
                //     }
                // }});

                return ok;
            }
        }
    </script>
</body>
</html>
```

