---
title: Vue DOM 更新完成后再执行函数
date: 2017-05-04 23:07:57
tags: [FE, Vue]
---

Vue 的数据变化后会更新 DOM，但只能保证在当前 tick 里面的代码全部执行完毕后更新 (事件队列)，不能保证数据一变化后就能用 document.querySelector() 立即获取到最新的 DOM。要保证在 DOM 更新以后执行某一块代码，就必须把这块代码放到下一次事件循环里面，比如 setTimeout(fn, 0)，这样 DOM 更新后，就会立即执行这块代码。

有些时候 DOM 更新完成后执行某些操作是有必要的，这时就可以使用 **Vue.nextTick()** 注册一个函数放到 Vue 的事件队列里，使其在下一个 tick 被执行。

![](/img/fe/fe-vue-next-tick.png)

例如使用 Vue + Semantic Ui 创建 Popup，新创建的 Popup 需要执行 popup() 后才会生效，此时在 DOM 更新完成后需要执行一下 popup() 函数。

一般有 3 种方式调用 **Vue.nextTick()**:

* 普通事件处理函数中，下面的 **[[1]]**，当有多个地方修改同一个变量时，每个地方都需要执行一次
* 监听指定的数据变化时，下面的 **[[2]]**，粒度细，只与数据是否变化有关，和修改数据的地方无关
* **updated()** 回调中，下面的 **[[3]]**，只要 DOM 变化了都会调用，无关的数据变化时都会调用，最省事，但是需要小心测试看看是否有副作用

<!--more-->

![](/img/fe/fe-vue-dom-updated-callback.png)

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title></title>
    <link rel="stylesheet" href="http://cdn.staticfile.org/semantic-ui/2.2.7/semantic.min.css">
    <script src="http://cdn.bootcss.com/jquery/1.9.1/jquery.min.js"></script>
    <script src="http://cdn.staticfile.org/semantic-ui/2.2.7/semantic.min.js"></script>
    <script src="http://cdn.staticfile.org/vue/2.0.3/vue.js"></script>
    <script src="http://cdn.staticfile.org/layer/2.3/layer.js"></script>

    <style media="screen">
        body {
            padding: 20px;
        }
    </style>
</head>

<body>
    <div id="vue-tags">
        <div class="ui button" @click="addTag()">Add Tag</div><br><br>

        <a class="ui tag label" :data-content="tag" v-for="tag in tags" v-html="tag"></a>
    </div>

    <script>
        var count = 3;
        new Vue({
            el: '#vue-tags',
            data: {
                tags: [1, 2]
            },
            methods: {
                addTag: function() {
                    this.tags.push(count++);

                    // [[1]] 事件处理函数中调用
                    // this.$nextTick() 也可以
                    // Vue.nextTick(function() {
                    //     $('.tag').popup({ position: 'bottom left' });
                    // });
                }
            },
            watch: {
                // [[2]] 指定变量数据变化时调用
                tags: function() {
                    Vue.nextTick(function() {
                        $('.tag').popup({ position: 'bottom left' });
                    });
                }
            },
            // [[3]] 只要 DOM 更新了就会调用
            updated: function() {
                // Vue.nextTick(function() {
                //     $('.tag').popup({ position: 'bottom left' });
                // });
            }
        });

        $('.tag').popup({ position: 'bottom left' });
    </script>
</body>

</html>
```

参考: [Vue nextTick 源码解读](http://www.tuicool.com/articles/QfMvAvB)

