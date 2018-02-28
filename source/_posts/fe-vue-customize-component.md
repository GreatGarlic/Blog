---
title: Vue 自定义组件
date: 2017-07-24 17:31:52
tags: [FE, Vue]
---

Vue 提供了自定义组件的功能，可以定义全局组件，也可以定义局部组件：

* 全局组件: 使用 **Vue.component()** 来注册
* 局部组件: 使用 Vue 对象的 **components** 属性来注册

下面先介绍全局组件的自定义，然后再简要的介绍局部组件的自定义。

<!--more-->

## 最简单的组件

定义:

```html
Vue.component('com-name', {
    template: '<span>你好</span>'
});
```

使用:

```html
<com-name></com-name>
```

组件的自定义和使用都非常简单，下面请看完整的可执行代码:

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <script src="http://cdn.bootcss.com/jquery/1.9.1/jquery.min.js"></script>
    <script src="http://cdn.staticfile.org/vue/2.0.3/vue.js"></script>
</head>

<body>
    <div id="app-one">
        <com-name></com-name>
    </div>

    <div id="app-two">
        <com-name></com-name>
    </div>

    <script>
        // 自定义全局组件
        // com-name 为组件的名字
        // template 为组件的模版
        Vue.component('com-name', {
            template: '<span>你好</span>'
        });

        new Vue({
            el: '#app-one',
            data: {}
        });
        new Vue({
            el: '#app-two',
            data: {}
        });
    </script>
</body>

</html>
```

输出:

```html
<div id="app-one">
    <span>你好</span>
</div>
<div id="app-two">
    <span>你好</span>
</div>
```

> 注: 下面为了描述简单，定义术语 parent 和 children:
>
> * `app-one`、`app-two` 称为 parent
> * `app-one`、`app-two` 调用的自定义组件 `com-name` 称为它们的 child

## 传递数据给组件

Parent 使用 **props** 给 children 组件传递数据:

```html
<body>
    <div id="app-one">
        <x-button :text="'No'"></x-button> <!-- 直接使用字符串 -->
        <x-button :text="buttonText"></x-button> <!-- 绑定 data -->
    </div>

    <script>
        Vue.component('x-button', {
            props: ['text'],
            template: '<button>{{text}}</button>'
        });

        new Vue({
            el: '#app-one',
            data: {
                buttonText: '按钮'
            }
        });
    </script>
</body>
```

> buttonText 的值改变后使用它的 x-button 的属性 text 的值也会变化。

## 使用 SLOT

上面使用 props 来给组件 x-button 传递按钮的文本，能不能这么 `<x-button>按钮</x-button>` 更直观的创建按钮呢？可以，使用 **SLOT** 就可以了：

```html
<body>
    <div id="app-one">
        <x-button>按钮</x-button>
    </div>

    <script>
        Vue.component('x-button', {
            template: '<button><slot></slot></button>'
        });

        new Vue({
            el: '#app-one'
        });
    </script>
</body>
```

输出:

```html
<button>按钮</button>
```

---

能不能使用多个 SLOT 呢？可以，有多个 SLOT 时，需要给每个 SLOT 一个名字:

```html
<body>
    <div id="app-one">
        <x-button>
            <span slot="first">前缀</span>
            <span slot="second">后缀</span>
        </x-button>
    </div>

    <script>
        Vue.component('x-button', {
            template: '<button><slot name="first"></slot> -按钮- <slot name="second"></slot></button>'
        });

        new Vue({
            el: '#app-one',
            data: {
                buttonText: '按钮'
            }
        });
    </script>
</body>
```

输出:

```html
<button><span>前缀</span> -按钮- <span>后缀</span></button>
```

## 组件发射信号

Parent 使用 `props` 和 `slot` 给 children 组件传递数据，children 使用 `emit` 把数据响应给 parent:

```html
<body>
    <div id="app-one">
        <x-button @increased="valueChanged"></x-button> - {{value}}
    </div>

    <script>
        // 点击按钮 count 加 1，让后发射信号，参数为当前的 count 值
        // parent 监听到 increased 信号，然后调用 valueChanged 函数
        Vue.component('x-button', {
            template: '<button @click="increase">Count - {{count}}</button>',
            data: function() {
                return {
                    count: 0
                };
            },
            methods: {
                increase: function() {
                    this.count += 1;
                    this.$emit('increased', this.count); // 发射 increased 信号，参数为 count
                }
            }
        });

        new Vue({
            el: '#app-one',
            data: {
                value: 0
            },
            methods: {
                valueChanged: function(c) {
                    this.value = c;
                }
            }
        });
    </script>
</body>
```

> 注意:
>
> * 组件的 data 必须是一个函数，返回一个 JSON 对象
> * 组件发射信号使用 `this.$emit` 把数据传给 parent
> * 信号的名字不能使用驼峰规则命名，例如 `valueChanged` 是无效的
> * 信号名字的单词间可以用 `-` 分割，例如 `value-changed` 是合法的

## 简化 template 模版

上面自定义组件的 template 是通过拼接字符串来实现的，如果这个字符串很长时就不好拼接了，可以把其放在 HTML 中的 **template** 元素里，通过 id 来引用，其他的 props, data, methods 等还是和以前的一样用法:

```html
<body>
    <div id="app-one">
        <x-button :text="'按钮'"></x-button> <!-- 绑定 data -->
    </div>

    <!-- 组件的 template 部分 -->
    <template id="x-button-template">
        <button @click="click">按钮</button>
    </template>

    <script>
        Vue.component('x-button', {
            template: '#x-button-template',
            methods: {
                click: function() {
                    console.log(new Date().getTime());
                }
            }
        })

        new Vue({
            el: '#app-one',
            data: {}
        });
    </script>
</body>
```

> 模版的标签名不一定要是 `template`，也可以是例如 `<script type="html/text">template content</script>`，只要是在网页中不可见的就好。

## 自定义局部组件

使用 Vue 对象的 **components** 属性来注册局部组件，下面定义的局部组件 **x-button** 只能在 **app-one** 中使用，不能在 **app-two** 使用:

```html
<body>
    <div id="app-one">
        <x-button :text="buttonText"></x-button>
    </div>
    <div id="app-two">
        <x-button :text="buttonText"></x-button> <!-- x-button 是在 app-one 的 Vue 对象中定义的，所以不能访问 -->
    </div>

    <script>
        new Vue({
            el: '#app-one',
            data: {
                buttonText: '按钮一'
            },
            components: {
                'x-button': {
                    props: ['text'],
                    template: '<button>{{text}}</button>'
                }
            }
        });

        new Vue({
            el: '#app-two',
            data: {
                buttonText: '按钮二'
            },
        });
    </script>
</body>
```