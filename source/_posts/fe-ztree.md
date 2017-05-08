---
title: zTree
date: 2017-04-14 13:51:34
tags: [FE, zTree]
---

> zTree is an advanced jQuery 'tree plug-in'. The performance is excellent, it is easy to configurw (with a full set of options), and has many advanced features (that usually only come with paid software).
>
> zTree is open source and uses the MIT license.

主页是 <http://www.treejs.cn>

<!--more-->

## 树形结构数据创建 zTree

树的父子关系使用节点的 children 数组来建立。

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>zTree</title>
    <link rel="stylesheet" href="http://cdn.staticfile.org/zTree.v3/3.5.28/css/zTreeStyle/zTreeStyle.min.css">
</head>

<body>
    <ul id="treeDemo" class="ztree"></ul>

    <script src="http://cdn.bootcss.com/jquery/3.2.1/jquery.min.js"></script>
    <script src="http://cdn.staticfile.org/zTree.v3/3.5.28/js/jquery.ztree.all.min.js"></script>
    <script>
        var setting = {};

        var nodes = [{
                name: '数学',
                open: false,
                children: [{
                    name: '参数方程'
                }, {
                    name: '贝塞尔曲线'
                }]
            },
            {
                name: '历史',
                open: true,
                children: [{
                    name: '玄武门事变'
                }, {
                    name: '桃园三结义'
                }]
            }
        ];

        var tree = $.fn.zTree.init($('#treeDemo'), setting, nodes);
    </script>
</body>

</html>
```

## 数组结构数据创建 zTree
数据库表中树形结构的存储一般都是通过 parentId 来建立数据之间的父子关系，zTree 也可以使用数组结构的数据，通过 parentId 来建立树节点之间的父子关系，需要设置 **setting.data.simple** 的属性来启用这种创建方式。

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>zTree</title>
    <link rel="stylesheet" href="http://cdn.staticfile.org/zTree.v3/3.5.28/css/zTreeStyle/zTreeStyle.min.css">
</head>

<body>
    <ul id="treeDemo" class="ztree"></ul>

    <script src="http://cdn.bootcss.com/jquery/3.2.1/jquery.min.js"></script>
    <script src="http://cdn.staticfile.org/zTree.v3/3.5.28/js/jquery.ztree.all.min.js"></script>
    <script>
        var setting = {
            data: {
                simpleData: {
                    enable : true,       // 启用数组结构的数据创建 zTree
                    idKey  : 'id',       // 定义 node id 的 key，可自定义
                    pIdKey : 'parentId', // 定义 parent id 的 key，可自定义
                    rootPId: 0           // 根节点的 parentId，这个节点是看不到的，因为不存在
                }
            }
        };

        // 节点的关系通过 parentId 来建立，顺序不重要
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

> 顶级节点（根节点）的 parentId 不一定要是 0，只要不是它的任意后代节点的 id 就可以，但是为了规范，最好还是定义一个根节点的 id。一般数据库的 id 都是从 1 开始，所以顶级节点的 parentId 都会设置为 0。
>
> zTree 允许存在多个根节点。

## 增删拖拽重命名

为了能够对 zTree 进行编辑，需要设置 **setting.edit.enable** 为 true，下面演示的功能有:

* 增加新的节点: `tree.addNodes(parentNode, index, newNodes, false)`，最后参数 false 表示展开
* 删除选中的节点: 删除前在回调函数 **beforeRemove()** 中先询问是否删除
* 重命名选中的节点: 重命名前在回调函数 **beforeRename()** 中先确定是否真的能够重命名
* 拖拽移动节点: 在 **beforeDrop()** 得不到新 parentId，所以需要去 **onDrop()** 中处理
* 取得选中的节点: **tree.getSelectedNodes()**

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>zTree</title>
    <link rel="stylesheet" href="http://cdn.staticfile.org/zTree.v3/3.5.28/css/zTreeStyle/zTreeStyle.min.css">
</head>

<body>
    <button id="add-node-button">添加节点</button>
    <ul id="treeDemo" class="ztree"></ul>

    <script src="http://cdn.bootcss.com/jquery/3.2.1/jquery.min.js"></script>
    <script src="http://cdn.staticfile.org/zTree.v3/3.5.28/js/jquery.ztree.all.min.js"></script>
    <script>
        var setting = {
            data: {
                simpleData: {
                    enable : true,       // 启用数组结构的数据创建 zTree
                    idKey  : 'id',       // 定义 node id 的 key，可自定义
                    pIdKey : 'parentId', // 定义 parent id 的 key，可自定义
                    rootPId: 0           // 根节点的 parentId，这个节点是看不到的，因为不存在
                }
            },
            edit: {
                enable: true // [*] 启用拖拽，重命名和删除等编辑操作
            },
            view: {
                selectedMulti: false, // [*] 为 true 时可选择多个节点，为 false 只能选择一个，默认为 true
            },
            callback: {
                onClick: function(event, treeId, treeNode) {
                    // [*] 点击节点的回调函数
                    // console.log(treeNode);
                },
                onDrop: function(event, treeId, treeNodes, targetNode, moveType, isCopy) {
                    // [*] 拖拽完成时的回调函数，此时 treeNodes 里的 parentId 已经是新 parent 的 id，
                    //     targetNode 有可能是 treeNodes 的新 parent，也有可能是 sibling
                    //     beforeDrop 中得不到新的 parentId，如果需要修改服务器上的 parentId 只好在 onDrop 里获取了
                    //     Ajax 同步请求服务器更新数据库中节点的 parentId，更像失败则弹出错误信息，提示用户刷新界面保证数据的正确显示
                    console.log(treeNodes[0]);
                },
                beforeRename: function(treeId, treeNode, newName, isCancel) {
                    // [*] 重命名前的回调函数，按下 ESC 取消重命名也会调用此函数
                    if (!isCancel) {
                        console.log(treeNode.id + ' : ' + newName);
                        // Ajax 同步请求服务器更新数据库中节点的名字，更新成功后返回 true，更新失败提示错误信息并返回 false
                        return true;
                    }
                },
                beforeRemove: function(treeId, treeNode) {
                    // [*] 删除节点前的回调函数
                    var ok = confirm('确定删除 ' +  treeNode.name + ' 吗？'); // 删除前最好是询问一下是否确定删除，防止误操作
                    if (!ok) { return false; }

                    // Ajax 同步请求服务器删除数据库中的节点，删除成功返回 true，删除失败提示错误信息并返回 false
                    return true;
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

        var newNodeId = 1000; // 新创建节点的 id

        // [*] 创建新节点
        $('#add-node-button').click(function(event) {
            var parentNode = getFirstSelectedNode(tree);
            var parentId   = (parentNode == null) ? 0 : parentNode.id;
            var node       = { 'id': newNodeId, 'parentId': parentId, 'name': 'Node-' + newNodeId};

            // Ajax 同步请求服务器在数据库中的创建新节点，创建成功则添加到 tree 里，创建失败则提示错误信息
            // [*] 创建新节点
            tree.addNodes(parentNode, -1, [node], false);

            newNodeId += 1;
        });

        /**
         * 取得树 zTree 选中的节点中的第一个 node，如果没有选中任何 node，则返回 null
         *
         * @param tree {zTreeObj} zTree 对象
         * @return 返回选中的 node，没有选中时返回 null
         */
        function getFirstSelectedNode(tree) {
            var selectedNodes = tree.getSelectedNodes();
            return selectedNodes.length == 0 ? null : selectedNodes[0];
        }
    </script>
</body>

</html>
```



## 数据操作

* 自定义的属性都会保存到 zTree 节点的数据中，也可以使用节点对象获取自定义属性
* **isParent** 决定一个节点是否以非叶子节点显示和操作，和节点下是否有子节点数据无关
* 修改节点 DOM 相关的数据后需要调用 **zTree.updateNode()** 更新节点
* 可以对节点的点击事件绑定回调函数

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>zTree</title>
    <link rel="stylesheet" href="http://cdn.staticfile.org/zTree.v3/3.5.28/css/zTreeStyle/zTreeStyle.min.css">
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
                    rootPId: 0
                }
            },
            callback: {
                // 节点点击时的事件处理
                onClick: function(event, treeId, treeNode) {
                    // [*] 获取节点的属性，treeNode 是一个 JSON 对象
                    console.log(treeNode.name);
                    console.log(treeNode.customField);

                    // [*] 修改节点数据后需要更新节点，例如叶子节点和非叶子节点互换
                    treeNode.isParent = !treeNode.isParent;
                    window.tree.updateNode(treeNode);
                }
            }
        };

        var nodes = [
            { 'id': 1, 'parentId': 0, 'name': '试卷', open: true },
            { 'id': 2, 'parentId': 1, 'name': '数学', customField: '数学-自定义属性' }, // [*] 自定义属性
            { 'id': 3, 'parentId': 1, 'name': '历史' },
            { 'id': 4, 'parentId': 2, 'name': '参数方程' },
            { 'id': 5, 'parentId': 2, 'name': '贝塞尔曲线' },
            { 'id': 6, 'parentId': 3, 'name': '玄武门事变' },
            { 'id': 7, 'parentId': 3, 'name': '桃园三结义', customField: '桃园三结义-自定义属性' }
        ];

        var tree = $.fn.zTree.init($("#treeDemo"), setting, nodes);
    </script>
</body>

</html>
```

## Ajax 加载节点数据

很多时候节点的数据并不是一次就全部准备好的，而是在展开非叶节点时使用 Ajax 去服务器端请求它的子节点，此时需要配置 **setting.async**:

* **url**: 请求的 URL
* **dataFilter**: 得到服务器响应时的回调函数，可以在此函数中对响应的数据进行处理。例如 Java Bean 中 boolean 属性名不要以 is 作为前缀，但是 zTree 使用属性 isParent 表示非叶子结点，所以在回调函数对服务器响应的数据进行处理是有必要的。

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>zTree</title>
    <link rel="stylesheet" href="http://cdn.staticfile.org/zTree.v3/3.5.28/css/zTreeStyle/zTreeStyle.min.css">
</head>

<body>
    <ul id="treeDemo" class="ztree"></ul>

    <script src="http://cdn.bootcss.com/jquery/3.2.1/jquery.min.js"></script>
    <script src="http://cdn.staticfile.org/zTree.v3/3.5.28/js/jquery.ztree.all.min.js"></script>
    <script>
        var setting = {
            data: {
                simpleData: {
                    enable: true,
                    idKey:  'id',
                    pIdKey: 'parentId'
                }
            },
            async: {
                enable: true,
                type: 'get',
                url: function(treeId, treeNode) {
                    // treeNode 不存在时表示需要加载根节点下的数据
                    return '/ztree/children?parentId=' + (treeNode ? treeNode.id : 0);
                },
                dataFilter: function(treeId, parentNode, children) {
                    if (children) {
                        for (var i = 0; i < children.length; i++) {
                            children[i].isParent = children[i].hasChildren;
                        }
                    }

                    return children;
                }
            },
            callback: {
                onClick: function(event, treeId, treeNode) {
                    console.log(treeNode.name);
                }
            }
        };

        var tree = $.fn.zTree.init($("#treeDemo"), setting);
    </script>
</body>

</html>
```

服务器端 TreeNode:

```java
package com.xtuer.bean;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class TreeNode {
    private int id;
    private int parentId;
    private String name;
    private boolean hasChildren;

    public TreeNode(int id, String name, boolean hasChildren, int parentId) {
        this.id = id;
        this.name = name;
        this.hasChildren = hasChildren;
        this.parentId = parentId;
    }
}
```

服务器端 Controller:

```java
@GetMapping("/ztree/children")
@ResponseBody
public List<TreeNode> ztreeChildren(@RequestParam int parentId) {
    List<TreeNode> children = new LinkedList<>();

    if (parentId == 0) {
        children.add(new TreeNode(1, "数学", true, parentId));
        children.add(new TreeNode(2, "历史", true, parentId));
    } else if (parentId == 1) {
        children.add(new TreeNode(11, "参数方程", false, parentId));
        children.add(new TreeNode(12, "贝塞尔曲线", false, parentId));
    } else if (parentId == 2) {
        children.add(new TreeNode(21, "玄武门事变", false, parentId));
        children.add(new TreeNode(22, "桃园三结义", false, parentId));
    }

    return children;
}
```

响应数据 JSON 格式:

```json
[{
    "id": 1,
    "parentId": 0,
    "name": "数学",
    "hasChildren": true
}, {
    "id": 2,
    "parentId": 0,
    "name": "历史",
    "hasChildren": true
}]
```

