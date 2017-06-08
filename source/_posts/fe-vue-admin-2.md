---
title: Vue 后台管理简单框架（二）
date: 2017-06-01 18:38:44
tags: [FE, Vue]
---

[Vue 后台管理简单框架（一）](/fe-vue-admin-1) 中搭建出了后台管理的页面框架，但是还没有添加其他的功能，例如使用 Vuex 实现模块间数据共享、与服务器通讯、功能独立为模块、修改打包选项等，这一章主要的内容就是介绍这些功能的实现。<!--more-->

## Vuex

**Vuex 是什么？**

Vuex 是一个专为 Vue.js 应用程序开发的**状态管理模式**。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。看不明白？其实就是保存数据的一个小工具，可以理解为和 localStorage 差不多，不过数据没有保存到硬盘上，一刷新页面就会没了。

**什么时候使用 Vuex?**

当我们的应用遇到**多个组件共享状态**时可以使用 Vuex，也就是多个组件共享变量时就可以使用 Vuex，共享数据变化时这些组件都同时更行。例如父子组建间通讯使用 props 和 emit，很多时候不够方便，如果这时使用 Vuex 的话，就会很简单（如果是提供给第三方使用的组件库的话，还是需要使用 props 和 emit）。

下面介绍 Vuex 的安装，注册和使用:

1. 安装 Vuex

   ```
   npm install vuex --save
   ```

2. 创建 src/store/index.js 文件，存储共享数据变量

   ```js
   import Vue from 'vue';
   import Vuex from 'vuex';

   Vue.use(Vuex);

   // state 中的 count 即是共享数据
   export default new Vuex.Store({
       state: {
           count: 10
       }
   });
   ```

3. main.js 中注册 Vuex 的 store

   ```js
   ...
   import store  from './store';

   new Vue({
       el: '#app',
       router,
       store,
       template: '<App/>',
       components: { App },
   });
   ```

   > new Vue() 中注册的 store, router 在 vue 文件中都可以使用 this 访问，例如 this.$store.state.count, this.$router.push('/hi')。

4. 使用 store 访问共享变量，例如在 Hello.vue 中

   ```js
   mounted() {
       this.$store.state.count += 1;
   }

   例如在其他 vue 文件中访问 count，因为我们意见引入了 Element，就用消息框吧:
   this.$message(`count value is ${this.$store.state.count}`);
   ```

## 与服务器通讯

vue-cli 推荐使用 Axios 与服务器通过 Ajax 通讯:

1. 安装 Axios

   ```
   npm install axios --save
   ```

2. 在需要使用的 vue 页面引入 Axios 就可以使用了

   ```js
   import axios from 'axios';

   axios.get('/rest', {params: {name: 'Biao'}}).then((result) => {
       console.log(result.data);
   });
   ```

不过，Axios 默认使用 application/json 并把参数放到 request body 中发送，标准的 RESTful 的方式，由于服务器端有时候从 request body 中取参数不方便，例如在 Spring MVC 中我个人更愿意使用 @RequestParam 获取参数，这种用法需要对 Axios 封装一下，或则使用 [jQuery 的 REST 插件](http://www.qtdebug.com/fe-rest/)。

## Ajax 跨域

> 配置 proxyTable 实现 Ajax 跨域。

使用 vue-cli 开发的时候，与服务器通讯是跨域的 Ajax，要么服务器允许跨域，要么 vue 中配置 proxyTable 让 vue 代理访问实现跨域:

* 服务器允许跨域，但是有一个缺点，Ajax 的 url 必须要带上服务器的 IP、域名和端口

* 配置 proxyTable 让 vue 代理访问实现跨域，Ajax 的 url 不需要带上服务器的 IP、域名和端口，修改 config/index.js 中的 dev.proxyTable

  ```js
  proxyTable: {
      '/api': {
          target: 'http://localhost:8080'
      }
  }
  ```

  这样我们在写url的时候，只用写成`/api/1`就可以代表 `http://localhost:8080/api/1`。

## 定义功能模块的模版

在一个大的 vue 页面中内容太多，可以把某些部分独立出来定义为一个小的模版，然后引用。例如 Big.vue 中有 10 个功能，全写在 Big.vue 中的话代码会比较多，模块不够清晰，可以把根据功能独立成模版，例如 module1.vue, module2.vue, ...，然后在 Big.vue 中使用 module1.vue, module2.vue 中定义的模版

```html
<!-- 文件名: module1.vue -->
<template>
    <div>Module 1</div>
</template>
<script>
    export default {

    };
</script>
```

```html
<!-- 文件名: Big.vue -->
<template>
    <div>
        <module1></module1> <!-- 使用模版 modul1 -->
    </div>
</template>
<script>
    import module1 from './module1';

    export default {
        components: {
            module1 // 注册模版 module1
        }
    };
</script>
```

> 模版之间的通讯可以使用 props + emit 或则上面的 vuex 的 store。

## 路由跳转

可以调用 router 的函数进行路由跳转

```js
this.$router.push('/');      // 跳转到首页
this.$router.push('/users'); // 跳转到 users 页面
this.$router.go(-1); // 后退到上一个页面
this.$route.path // 取得当前页面的路径，例如 /users
```

## 加载数据

> mounted() 中从服务器加载数据。

一般应该在 mounted() 事件中加载数据，不应该在 created() 中，因为 created() 时 el 还没有被挂载，mounted() 时 el 被替换为 $el，数据就可以显示到界面上，而且在页面中只执行一次。下面以加载用户数据显示到 table 中为例，加载数据时显示 loading 状态，加载完成后用户数据显示到 table 中，loading 状态消失:

```html
<template>
    <!-- v-loading 为 true 时显示 loading 状态 -->
    <el-table :data="tableData" border style="width: 100%" v-loading="loading">
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
                loading: true,
                tableData: []
            };
        },
        methods: {
            formatter(row, column) {
                return row.address;
            },
            filterTag(value, row) {
                return row.tag === value;
            }
        },
        mounted() {
            this.$message(`count value is ${this.$store.state.count}`);
            // 模拟从服务器请求数据需要 1 秒
            setTimeout(() => {
                this.loading = false;
                this.tableData = [{
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
                }];
            }, 1000);
        }
    };
</script>
```

## Build 配置

执行 npm run build 编译出发布的文件，默认中 dist 文件夹中，context path 为 /，但是默认的配置不一定符合项目的需求。

例如在 Spring MVC 中，context path 很多时候不是 /，而是项目名，vue 中的 index.html 要重命名为 admin.html 并且放到 static/html 目录下，通过配置 mvc:resources 来访问，如 http://host/html/admin.html，这些都可以通过配置 config/index.js 中的 build 来实现，例如:

```js
build: {
    env: require('./prod.env'),
    index: path.resolve(__dirname, '../dist/static/html/admin.html'),
    assetsRoot: path.resolve(__dirname, '../dist'),
    assetsSubDirectory: 'static',
    assetsPublicPath: '/fox',
    ...
```

* index: 生成的页面文件的路径
* assetsRoot: 编译输出的目录
* assetsSubDirectory: 编译输出的 js、img、css 等文件的文件夹名字
* assetsPublicPath: context path

## 引入静态文件

> 使用 ESJ 在 index.html 中判断不同的环境下使用不同的路径引入静态文件。

我们自己的静态文件 js、css、image 等都会放到 static 目录下（这里的文件 webpack 不会进行压缩打包），然后在 index.html 中引入

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>vue-admin</title>
</head>

<body>
    <div id="app"></div>
    <!-- built files will be auto injected -->

    <script src="/static/lib/jquery.min.js" charset="utf-8"></script>
    <script src="/static/lib/jquery.rest.js" charset="utf-8"></script>
</body>

</html>

```

如果项目的 context path 是 **/fox**，修改 build.assetsPublicPath 为 **/fox**，但是编译后 js 引入部分仍然为:

```html
<script src="/static/lib/jquery.min.js" charset="utf-8"></script>
<script src="/static/lib/jquery.rest.js" charset="utf-8"></script>
```

实际需要:

```html
<script src="/fox/static/lib/jquery.min.js" charset="utf-8"></script>
<script src="/fox/static/lib/jquery.rest.js" charset="utf-8"></script>
```

尝试修改 config/index.js 中 dev.assetsPublicPath 为 **/fox**，`npm run dev` 项目启动后却无法访问，可能是不能修改这个选项吧。考虑到可以使用 EJS 模版，于是修改 index.html 通过 if else 判断在开发环境和编译时使用不同的路径引入静态文件:

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>vue-admin</title>
</head>

<body>
    <div id="app"></div>
    <!-- built files will be auto injected -->

    <% if (process.env.NODE_ENV === 'production') {%>
        <script src="/fox/static/lib/jquery.min.js" charset="utf-8"></script>
        <script src="/fox/static/lib/jquery.rest.js" charset="utf-8"></script>
    <%} else {%>
        <script src="/static/lib/jquery.min.js" charset="utf-8"></script>
        <script src="/static/lib/jquery.rest.js" charset="utf-8"></script>
    <%}%>
</body>

</html>
```

再次编译后看到引入静态文件时加上了 context path。

## 登陆

登陆就不在此 SPA 前端页面中做了，因为一个项目里不只是这一个单页面，所以可以由服务器端的 Spring Security 来控制权页面的限访问，还可以方便的实现各种登陆方式、remember me、验证码等，提供接口获取当前登陆用户的信息给前端 vue 使用，然后 vue 根据用户的权限信息显示不同的菜单。

可以在 main.js 中实现 router 的 **beforeEach()** 勾子函数，每次路由的时候使用 Ajax 在其中请求当前登陆用户的信息并保存到 store 中，然后 Home.vue 中根据用户的权限过滤显示菜单。

```js
// 每次路由都请求一下登陆用户信息
router.beforeEach((to, from, next) => {
    // 同步使用 Ajax 调用接口获取登陆用户的信息
    const loginUser = {id: 123, name: 'Alice', logined: true};

    if (loginUser.logined) {
        next();
    } else {
        // 没有登陆则访问登陆页面
        window.location.href = 'http://localhost:8080/login';
    }
});

new Vue({
	...
    router,
    ...
});
```

> 有可能你要问，每次路由都要请求一次登陆用户信息，效率是不是不高，能不能访问一次使用 sessionStorage 存储起来？
>
> 一般一个系统不可能就只有一个地方能够注销，例如:
>
> * 点击页面 A、B、C 上的注销按钮进行注销
> * 直接访问注销的 URL 进行注销
>
> 从其他地方注销后需要被监测到，把用户信息从 sessionStorage 里删除，关键就是怎么被监测到。使用 Websocket，定时轮训等？那还不如每次路由时请求一次呢，何况 Ajax 是很快的。

## 懒加载

打包后页面的 js 全打包到了 app.js 中，如果页面的模块非常多导致 app.js 很大，可以把每个模块的 js 单独打包，在访问具体模块的时候才加载它的 js，只需要在 router 引入 vue 时使用 require 加载即可:

```js
// 非懒加载
// import NotFound from '@/view/404';
// import Home  from '@/view/Home';
// import Users from '@/view/information/Users';
// import Hello from '@/view/information/Hello';
// import Ajax  from '@/view/information/Ajax';

// 懒加载
const NotFound = resolve => require(['@/view/404'], resolve);
const Home  = resolve => require(['@/view/Home'], resolve);
const Users = resolve => require(['@/view/information/Users'], resolve);
const Hello = resolve => require(['@/view/information/Hello'], resolve);
const Ajax  = resolve => require(['@/view/information/Ajax'], resolve);

Vue.use(Router);

const router = new Router({
    routes: [{
        path: '/',
        component: Home,
        children: [
            // children 下的 component 会在 Home 中的 <router-view> 生成
            { path: '/users', component: Users, name: 'users' },
            { path: '/hello', component: Hello, name: 'hello' },
            { path: '/ajax', component: Ajax, name: 'ajax' }
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

## 参考资料

* [Vuex](https://vuex.vuejs.org/zh-cn/intro.html)
* [Axios](https://www.npmjs.com/package/axios)
* [Axios 在 vue 中的简单配置与使用](http://blog.csdn.net/sinat_17775997/article/details/69367204)
* [proxyTable 解决开发环境的跨域问题](http://www.jianshu.com/p/95b2caf7e0da)
* [懒加载](https://router.vuejs.org/zh-cn/advanced/lazy-loading.html)