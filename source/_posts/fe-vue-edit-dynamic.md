---
title: Vue 动态显示编辑按钮和计算 Class
date: 2017-04-28 10:29:17
tags: FE
---

下面介绍使用 vue 

* 动态的显示编辑按钮

  当鼠标移动到 segment 上时显示编辑按钮(**@mouseenter** 事件)，鼠标离开 segment 时隐藏编辑按钮(**@mouseleave** 事件)，需要定义一个属性如 editable 表示当前 segment 是否可编辑，使用 **v-show** 进行判断显示和隐藏

* 动态的计算 class

  复杂情况下不同元素的 class 不一样，此时可以使用函数动态计算 class，在 **:class** 中使用此函数，不同的参数计算出来的 class 不一样

![](/img/fe/vue-edit-dynamic.png)<!--more-->

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

        #vue-segments {
            width: 350px;
        }

        .icon.edit,
        .icon.delete {
            float: right;
        }

        .icon.edit:hover,
        .icon.delete:hover {
            color: darkred;
        }
    </style>
</head>

<body>
    <div id="vue-segments">
        <div class="ui segment" :class="segmentClass(index)" v-for="(item, index) in items" @mouseenter="item.editable=true" @mouseleave="item.editable=false">
            <i class="icon delete" v-show="item.editable" @click="remove(index)"></i>
            <i class="icon edit" v-show="item.editable" @click="edit(index)"></i>
            <span class="text" v-html="item.text"></span>
        </div>
    </div>

    <script src="http://cdn.bootcss.com/jquery/1.9.1/jquery.min.js"></script>
    <script src="http://cdn.staticfile.org/vue/2.0.3/vue.js"></script>
    <script src="http://cdn.staticfile.org/semantic-ui/2.2.7/semantic.min.js"></script>
    <script src="http://cdn.staticfile.org/layer/2.3/layer.js"></script>
    <script>
        new Vue({
            el: '#vue-segments',
            data: {
                items: [{
                    text: '<i class="icon tag"></i> This segment is on top',
                    editable: false
                }, {
                    text: 'This method is a shortcut for .on( "mouseenter", handler ) in the first two variations, and .trigger( "mouseenter" ) in the third.',
                    editable: false
                }, {
                    text: 'A function to execute each time the event is triggered.',
                    editable: false
                },{
                    text: 'This segment is on bottom',
                    editable: false
                }],
                currentIndex: -1
            },
            methods: {
                // 动态计算 segment 的 class
                segmentClass: function(index) {
                    if (index === 0) {
                        return 'ui top attached segment';
                    } else if (index === this.items.length - 1) {
                        return 'ui bottom attached segment';
                    } else {
                        return 'ui attached segment';
                    }
                },
                edit: function(index) {
                    layer.msg('编辑: ' + this.items[index].text);
                },
                remove: function(index) {
                    layer.msg('删除: ' + this.items[index].text);
                }
            }
        });
    </script>
</body>

</html>
```

