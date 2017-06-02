---
title: Vue 后台管理端简单框架（一）
date: 2017-05-30 16:03:34
tags: [FE, Vue]
---

[vue-cli 简单搭建项目框架](/fe-vue-skeleton/) 后我们知道了怎么使用 **vue-cli** 创建一个项目，但是页面比较简单，一般管理功能的界面如下:

![](/img/fe/vue-admin-ui.png)

下面介绍怎么实现一个这样的界面，UI 的框架使用 [Element](http://element.eleme.io)，命令行进入前面创建的项目目录，安装 Element:

```
npm install element-ui --save
```

然后在 main.js 中引入 Element 就能使用 Element 了，具体参考 main.js:

```js
import Element from 'element-ui';
import 'element-ui/lib/theme-default/index.css';

Vue.use(Element);
```

<!--more-->

项目的结构如下:

![](/img/fe/vue-admin-files.png)

为了统一管理页面的 vue 文件，把他们都放到 src/view 目录中，最好还要根据左边侧边栏的结构把 vue 文件也分子目录存放，这样更便于管理。Home.vue 是管理功能的界面的主要结构文件，404.vue 在找不到页面时显示。需要的文件都在下面列出来的，复制到项目即可:

* main.js
* router/index.js
* App.vue
* Home.vue
* 404.vue
* Users.vue (页面样例)
* Hello.vue (页面样例)

## main.js

```js
// The Vue build version to load with the `import` command
// (runtime-only or standalone) has been set in webpack.base.conf with an alias.
import Vue from 'vue';
import Element from 'element-ui';
import 'element-ui/lib/theme-default/index.css';

import App from './App';
import router from './router';

Vue.use(Element);
Vue.config.productionTip = false;

new Vue({
    el: '#app',
    router,
    template: '<App/>',
    components: { App },
});
```

> 在 main.js 中引入 Element，例如 vuex 也在此引入。

## router/index.js

```js
import Vue from 'vue';
import Router from 'vue-router';

import NotFound from '@/view/404';
import Home  from '@/view/Home';
import Users from '@/view/information/Users';
import Hello from '@/view/information/Hello';

Vue.use(Router);

export default new Router({
    routes: [{
        path: '/',
        component: Home,
        children: [
            // children 下的 component 会在 Home 中的 <router-view> 生成
            { path: '/users', component: Users, name: 'users' },
            { path: '/hello', component: Hello, name: 'hello' }
        ]
    }, {
        path: '/404',
        component: NotFound,
        hidden: true
    }, {
        path: '*',
        hidden: true,
        redirect: { path: '/404' }
    }]
});
```

> 配置页面的路由，注意 Home 下有 children，则说明在 Home 页面中进行路由的页面将替换 Home 中的  router-view，而不是 App 中的 router-view。
>
> 此外还配置了 404 的路由。

## App.vue

```html
<template>
    <div id="app">
        <transition name="fade" mode="out-in">
            <router-view></router-view>
        </transition>
    </div>
</template>

<script>
    export default {};
</script>

<style lang="scss">
    html, body {
        font-family: '微软雅黑', Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        margin: 0px;
        padding: 0px;
        font-size: 14px;
    }

    #app {
        position: absolute;
        top: 0;
        left: 0;
        bottom: 0;
        right: 0;
    }

    /* transition fade 的样式 */
    .fade-enter-active,
    .fade-leave-active {
    	transition: all .2s ease;
    }
    .fade-enter,
    .fade-leave-active {
    	opacity: 0;
    }
</style>
```

> Home, 404 等都是替换此页中的 router-view，因为它们在 routes 配置中是第一级 path。
>
> 动画 transition 中的 mode="out-in" 不能省略，动画的效果也需要自己用 css 定义。

## Home.vue

Home.vue 主要实现管理端页面的结构，左边是菜单侧边导航栏，中间上部是登陆用户的信息，中间主要部分是管理功能的显示区。

```html
<template>
    <div class="container">
        <!-- 侧边导航栏 -->
        <div class="sidebar">
            <el-menu :default-active="$route.path" router theme="dark">
                <el-submenu index="1">
                    <template slot="title"><i class="el-icon-message"></i>信息管理</template>
                    <el-menu-item index="/users">用户</el-menu-item>
                    <el-menu-item index="/hello">Hello</el-menu-item>
                    <el-menu-item index="1-3">选项3</el-menu-item>
                    <el-menu-item index="1-4-1">选项1</el-menu-item>
                </el-submenu>
                <el-menu-item index="2"><i class="el-icon-menu"></i>导航二</el-menu-item>
                <el-menu-item index="3"><i class="el-icon-setting"></i>导航三</el-menu-item>
            </el-menu>
        </div>

        <!-- 主体部分 -->
        <div class="main">
            <!-- 头部 -->
            <el-row class="header" type="flex" justify="space-between">
                <el-col :span="10" class="logo"></el-col>
                <el-col :span="4" class="userinfo">
                    <el-dropdown trigger="hover">
                        <span class="el-dropdown-link userinfo-inner"><img class="avatar" :src="avatar">{{username}}</span>
                        <el-dropdown-menu slot="dropdown">
                            <el-dropdown-item>消息</el-dropdown-item>
                            <el-dropdown-item>设置</el-dropdown-item>
                            <el-dropdown-item divided @click.native="logout">退出登录</el-dropdown-item>
                        </el-dropdown-menu>
                    </el-dropdown>
                </el-col>
            </el-row>

            <!-- 内容区 -->
            <el-row>
                <el-col :span="24" class="content">
                    <transition name="fade" mode="out-in">
                        <router-view></router-view>
                    </transition>
                </el-col>
            </el-row>
        </div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                systemName: '管理系统',
                username: '公孙二狗',
                avatar: '/static/img/avatar.jpg'
            };
        },
        methods: {
            // 退出登录
            logout() {
                this.$confirm('确认退出吗?', '提示', {}).then(() => {}).catch(() => {});
            }
        }
    };
</script>

<style lang="scss">
    $sidebarWidth: 200px; // 侧边栏的宽
    $headerHeight: 30px; // 头部的高

    .container {
        display: flex;
        position: absolute;
        top: 0;
        left: 0;
        bottom: 0;
        right: 0;

        .sidebar {
            width: $sidebarWidth;
            left: 0;
            top: 0;
            bottom: 0;
            background: #334156;
            overflow: auto;
        }

        .main {
            overflow: hidden;
            flex: 1;

            .header {
                display: flex;
                height: $headerHeight;
                line-height: $headerHeight;
                padding: 0 10px;
                margin-bottom: 0;
                background: #334156;

                .userinfo {
                    text-align: right;

                    .userinfo-inner {
                        cursor: pointer;
                        color: #BDC9D6;
                        font-size: 12px;

                        .avatar {
                            width: 20px;
                            height: 20px;
                            border-radius: 20px;
                            margin: 5px 0 0px 5px;
                            float: right;
                            box-shadow: 0 0 5px rgba(0, 0, 0, 0.5);
                        }
                    }
                }
            }

            .content {
                padding: 10px;
            }
        }
    }
</style>
```

> Home 实现了管理界面的结构，点击导航栏中的项时，对应路由的页面显示在页面的主要内容区。
>
> Home 主要是用了 Element 的 NavMenu，dropdown 和 layout，同时也用了 flex，具体的用法需要去阅读相关帮助文档。
>
> `<el-menu :default-active="$route.path" router>` 中的 `:default-active="$route.path"` 比较有意思，页面刷新或者返回时，自动的展开上一个页面时侧边栏，`$route.path` 值例如为 `/users`。

## 404.vue

```html
<template>
    <div id="not-found">
        <h1>Page Not Found</h1>
        <p>Sorry, but the page you were trying to view does not exist.</p>
    </div>
</template>

<style lang="scss">
    #not-found {
        display: table;
        width: 100%;
        height: 100%;
        color: #888;
        margin: 0 auto;
        padding-top: 20px;
        text-align: center;

        h1 {
            color: #555;
            font-size: 2em;
            font-weight: 400;
        }

        @media only screen and (max-width: 280px) {
            p {
                width: 95%;
            }

            h1 {
                font-size: 1.5em;
                margin: 0 0 0.3em;
            }
        }
    }
</style>
```

## Users.vue

用户信息的表格，为了展示路由效果的的同时展示一下 Element 的 table 的用法，很简洁，自己实现同样的效果就不只这点代码了，文档请参考 <http://element.eleme.io/#/zh-CN/component/table>

```html
<template>
    <el-table :data="tableData" border style="width: 100%">
        <el-table-column prop="date" label="日期" sortable width="180"></el-table-column>
        <el-table-column prop="name" label="姓名" width="180"></el-table-column>
        <el-table-column prop="address" label="地址" :formatter="formatter"></el-table-column>
        <el-table-column prop="tag" label="标签" width="100" :filters="[{ text: '家', value: '家' }, { text: '公司', value: '公司' }]" :filter-method="filterTag" filter-placement="bottom-end">
            <template scope="scope">
                <el-tag :type="scope.row.tag === '家' ? 'primary' : 'success'" close-transition>{{scope.row.tag}}</el-tag>
            </template>
        </el-table-column>
    </el-table>
</template>

<script>
    export default {
        data() {
            return {
                tableData: [{
                    date: '2016-05-02',
                    name: '王小虎',
                    address: '上海市普陀区金沙江路 1518 弄',
                    tag: '家'
                }, {
                    date: '2016-05-04',
                    name: '王小虎',
                    address: '上海市普陀区金沙江路 1517 弄',
                    tag: '公司'
                }, {
                    date: '2016-05-01',
                    name: '王小虎',
                    address: '上海市普陀区金沙江路 1519 弄',
                    tag: '家'
                }, {
                    date: '2016-05-03',
                    name: '王小虎',
                    address: '上海市普陀区金沙江路 1516 弄',
                    tag: '公司'
                }]
            };
        },
        methods: {
            formatter(row, column) {
                return row.address;
            },
            filterTag(value, row) {
                return row.tag === value;
            }
        }
    };
</script>
```

## Hello.vue

普通页面，随便写点就行，只是为了展示路由效果的。

```html
<template>
    <div class="hello">
        <h1>{{ msg }}</h1>
        <h2>Essential Links</h2>
        <ul>
            <li><a href="https://vuejs.org" target="_blank">Core Docs</a></li>
            <li><a href="https://forum.vuejs.org" target="_blank">Forum</a></li>
            <li><a href="https://gitter.im/vuejs/vue" target="_blank">Gitter Chat</a></li>
            <li><a href="https://twitter.com/vuejs" target="_blank">Twitter</a></li>
            <br>
            <li><a href="http://vuejs-templates.github.io/webpack/" target="_blank">Docs for This Template</a></li>
        </ul>
        <h2>Ecosystem</h2>
        <ul>
            <li><a href="http://router.vuejs.org/" target="_blank">vue-router</a></li>
            <li><a href="http://vuex.vuejs.org/" target="_blank">vuex</a></li>
            <li><a href="http://vue-loader.vuejs.org/" target="_blank">vue-loader</a></li>
            <li><a href="https://github.com/vuejs/awesome-vue" target="_blank">awesome-vue</a></li>
        </ul>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                msg: 'Welcome to Your Vue.js App',
            };
        },
    };
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    h1,
    h2 {
        font-weight: normal;
    }

    ul {
        list-style-type: none;
        padding: 0;
    }

    li {
        display: inline-block;
        margin: 0 0px;
    }

    a {
        color: #42b983;
    }
</style>
```

