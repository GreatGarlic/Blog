---
title: Vue Todo
date: 2017-04-24 20:50:24
tags: FE
---

实现一个简单的 Todo 来介绍不同的 Vue 对象共享同一个 data 数组、Todo 的增加、删除、编辑:

![](/img/fe/vue-todo.png)

左边的 Todo 和右边的 Dropdown 分别用一个 Vue 对象来渲染，它们共享使用同一个数组 todos，当左边的 Todo 修改了 todos 的数据后，右边的 Dropdown 的数据也会同时自动更新。点击编辑按钮，在 todo 原来的地方显示一个 input 进行编辑。<!--more-->

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

        #vue-todos .todo input {
            display: none;
        }
      
        #vue-todos .todo.editing input {
            display: block;
        }

        #vue-todos .todo.editing span {
            display: none;
        }

        #vue-todos, #vue-dropdown {
            float: left;
            margin-left: 20px;
        }
    </style>
</head>
<body>
    <div id="vue-todos" class="ui form" style="width: 300px;">
        <table class="ui celled striped table">
            <tbody>
                <!-- 每个 todo 一行 -->
                <tr class="todo" :class="{editing: editedTodo==todo}" v-for="todo in todos" :key="todo.id" :data-id="todo.id">
                    <td @dblclick="editTodo(todo)">
                        <span class="name">{{todo.name}}</span>
                        <input type="text" v-model.trim="todo.name" @keyup.enter="updateTodo(todo)" @keyup.esc="cancelEdit()" @blur="cancelEdit()" v-todo-focus="todo==editedTodo">
                    </td>
                    <td class="collapsing">
                        <!-- 编辑和删除按钮 -->
                        <i class="icon edit"   @click="editTodo(todo)"></i>
                        <i class="icon delete" @click="removeTodo(todo)"></i>
                    </td>
                </tr>
            </tbody>
            <tfoot class="full-width">
                <tr>
                    <!-- 添加的按钮 -->
                    <th colspan="2"><i class="plus icon" style="float: right;" @click="createTodo()"></i></th>
                </tr>
            </tfoot>
        </table>
    </div>

    <div id="vue-dropdown" class="ui floating labeled icon dropdown button">
        <i class="world icon"></i>
        <span class="text">Select Todo</span>
        <div class="menu">
            <div class="item" v-for="todo in todos">{{todo.name}}</div>
        </div>
    </div>

    <script src="http://cdn.bootcss.com/jquery/1.9.1/jquery.min.js"></script>
    <script src="http://cdn.staticfile.org/semantic-ui/2.2.7/semantic.min.js"></script>
    <script src="http://cdn.staticfile.org/vue/2.0.3/vue.js"></script>
    <script>
        // 2 个 Vue 对象共享的数组
        var todos = [
            {id: 1, name: '后天考试'},
            {id: 2, name: '出差上海'}
        ];

        var vueDropdown = new Vue({
            el: '#vue-dropdown',
            data: {
                todos: todos
            }
        });

        var vueTodo = new Vue({
            el: '#vue-todos',
            data: {
                todos: todos,
                editedTodo: null, // 当前编辑的 todo
                cachedName: ''    // 缓存需要编辑的内容，这里为 todo.name
            },
            methods: {
                createTodo: function() {
                    // TODO: 发送请求到服务器上创建 todo，然后更新到 todos
                    this.todos.push({id: 1, name: 'New Todo'});
                },
                updateTodo: function(todo) {
                    // TODO: 发送请求到服务器上更新 todo，然后更新到 todos
                    this.editedTodo = null;
                },
                removeTodo: function(todo) {
                    // TODO: 发送请求到服务器先删除数据，然后再从 todos 里删除
                    this.todos.splice(this.todos.indexOf(todo), 1);
                },
                editTodo: function(todo) {
                    this.editedTodo = todo;
                    this.cachedName = todo.name;
                },
                cancelEdit: function() {
                    this.editedTodo.name = this.cachedName;
                    this.editedTodo = null;
                }
            },
            // A custom directive to wait for the DOM to be updated.
            directives: {
                'todo-focus': function(el, binding) {
                    if (binding.value) {
                        el.focus()
                    }
                }
            }
        });

        $('.dropdown').dropdown(); // 在 vueDropdown 后初始化，因为 Vue 会修改 dropdown 的结构
    </script>
</body>
</html>
```

> 数组的操作需要使用 pop, push, splice 等 Vue 才会自动更新 Dom，使用下标操作则不会更新 Dom。
>
> ## 数组先赋值为 []，然后重新赋值为新数组也会更新 DOM。

## 思考

数据只有存储到服务器上才有意义，所以上面的代码中注释为 TODO 的地方就是需要和服务器交互的地方，实际项目里需要使用 Ajax 把数据存储到服务器。