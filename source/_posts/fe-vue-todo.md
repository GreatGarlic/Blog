---
title: Vue Todo
date: 2017-04-24 20:50:24
tags: FE
---

实现一个简单的 Todo 来介绍不同的 Vue 对象共享同一个 data 数组、Todo 的增加、删除、编辑:

![](/img/fe/vue-todo.png)

左边的 Todo 和右边的 Dropdown 分别用一个 Vue 对象来渲染，它们共享使用同一个数组 items，当左边的 Todo 修改了 items 的数据后，右边的 Dropdown 的数据也会同时自动更新。点击编辑按钮，在 item 原来的地方显示一个 input 进行编辑。<!--more-->

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title></title>
    <link rel="stylesheet" href="http://cdn.staticfile.org/semantic-ui/2.2.7/semantic.min.css">
    <style media="screen">
        body {
            padding: 20px;
        }

        .icon:hover {
            color: red;
        }

        .ui.table {
            margin: 0;
        }
        #vue-todo input {
            display: none;
        }

        #vue-todo, #vue-dropdown {
            float: left;
            margin-left: 20px;
        }
    </style>
</head>

<body>
    <div id="vue-todo" class="ui form" style="width: 300px;">
        <table class="ui celled striped table">
            <tbody>
                <!-- 每个 item 一行 -->
                <tr v-for="(item, index) in items" :key="item.id" :data-id="item.id">
                    <td>
                        <span class="name">{{item.name}}</span>
                    </td>
                    <td class="collapsing">
                        <!-- 编辑和删除按钮 -->
                        <i class="icon edit"   @click="showInput(index, $event)"></i>
                        <i class="icon delete" @click="removeItem(index)"></i>
                    </td>
                </tr>
            </tbody>
            <tfoot class="full-width">
                <tr>
                    <!-- 添加的按钮 -->
                    <th colspan="2"><i class="plus icon" style="float: right;" @click="createItem()"></i></th>
                </tr>
            </tfoot>
        </table>
        <!-- 输入框 -->
        <input class="ui input" type="text" @keyup.enter="updateItem()" @blur="hideInput()" @keyup.esc="hideInput()">
    </div>

    <div id="vue-dropdown" class="ui floating labeled icon dropdown button">
        <i class="world icon"></i>
        <span class="text">Select Item</span>
        <div class="menu">
            <div class="item" v-for="item in items">{{item.name}}</div>
        </div>
    </div>

    <script src="http://cdn.bootcss.com/jquery/1.9.1/jquery.min.js"></script>
    <script src="http://cdn.staticfile.org/semantic-ui/2.2.7/semantic.min.js"></script>
    <script src="http://cdn.staticfile.org/vue/2.0.3/vue.js"></script>
    <script>
        // 2 个 Vue 对象共享的数组
        var items = [
            {id: 1, name: '后天考试'},
            {id: 2, name: '出差上海'}
        ];

        var vueDropdown = new Vue({
            el: '#vue-dropdown',
            data: {
                items: items
            }
        });

        var vueTodo = new Vue({
            el: '#vue-todo',
            data: {
                items: items,
                currentIndex: -1 // 正在编辑的 item 的下标
            },
            methods: {
                showInput: function(index, event) {
                    this.currentIndex = index;

                    var $input = this.getInput();
                    $input.siblings('.name').show(); // 显示当前 input 所在行的 name

                    // 找到要插入的行，隐藏 name，然后插入 input
                    $td = $(event.target).parent().prev();
                    $('.name', $td).hide();
                    $input.val(items[index].name).appendTo($td).show().focus();
                },
                hideInput: function() {
                    // 隐藏 input，把相应的 name 显示出来，然后把 input 从 item 中移走，防止 item 被删除时 input 也被删除
                    var $input = this.getInput();
                    $input.hide();
                    $input.siblings('.name').show();
                    $input.appendTo('#vue-todo');
                },
                createItem: function() {
                    // TODO: 发送请求到服务器上创建 item，然后更新到 items
                    window.newId = window.newId || 3;
                    items.push({id: newId, name: 'New-' + newId++});
                },
                updateItem: function() {
                    // TODO: 发送请求到服务器上更新 item，然后更新到 items
                    var $input = this.getInput();
                    items[this.currentIndex].name = $input.val();
                    items.splice(this.currentIndex, 1, items[this.currentIndex]);

                    $input.siblings('.name').show();
                    this.hideInput();
                },
                removeItem: function(index) {
                    // TODO: 发送请求到服务器先删除数据，然后再从 items 里删除
                    items.splice(index, 1);
                },
                getInput: function() {
                    return $('#vue-todo input'); // 返回 input
                }
            }
        });

        $('.dropdown').dropdown(); // 在 vueDropdown 后初始化，因为 Vue 会修改 dropdown 的结构
    </script>
</body>

</html>
```

> 数组的操作需要使用 pop, push, splice 等 Vue 才会自动更新 Dom，使用下标操作则不会更新 Dom。

## 思考

数据只有存储到服务器上才有意义，所以上面的代码中注释为 TODO 的地方就是需要和服务器交互的地方，实际项目里需要使用 Ajax 把数据存储到服务器。