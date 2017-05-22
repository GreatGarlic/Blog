---
title: vue-cli 简单搭建项目框架
date: 2017-05-19 08:41:14
tags: [FE, Vue]
---

使用 vue-cli 生成项目框架后，一般都会根据自己或则团队的风格微调一些参数。

## 具体步骤

1. 安装 node

2. 安装 vue-cli

   ```
   npm install -g vue-cli
   ```

3. 创建项目: 

   ```
   vue init webpack projectname
   ```
   <!--more-->
   > 注意: 
   >
   > * 项目名称不能有大写字母
   > * ESLint 使用 Airbnb 风格会更合适一些
   > * 这里不安装 unit tests 和 e2e tests
   >
   > ![](/img/fe/vue-cli-1.png)

4. 安装项目需要的所有插件

   ```
   cd projectname
   npm install
   ```

5. 支持 SCSS

   ```
   npm install --save-dev node-sass
   npm install --save-dev sass-loader
   ```

6. ESHint 设置，修改 .eshintrc.js 的 rules，去掉一些严格的检查

   ```
   'indent': 0,
   'no-console': 0,
   'no-undef': 0,
   'no-trailing-spaces': 0,
   ```

   > 我喜欢使用 4 个空格来缩进，Airbnb 默认是 2 个，很多前端的都喜欢使用 2 个进行缩进，为了简单起见，不让空格的个数造成编译时错误，关闭掉它即可。

7. 编辑器使用 4 个空格缩进，修改 **.editorcongif**

   ```
   indent_size = 4
   ```

8. 图片不使用 Base64 的字符串嵌入到网页里，修改 **build/webpack.base.conf.js**

   ```
   {
       test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
       loader: 'url-loader',
       options: {
           limit: 1,
           name: utils.assetsPath('img/[name].[hash:7].[ext]')
       }
   }
   ```

   > limit 表示图片大小小于它的图片将使用 Base64 的方式嵌入网页。

9. 修改端口号

   默认是 8080，Tomcat 默认也是使用 8080，为了方便开发，修改 config/index.js 中的 dev.port 为 8888

10. 执行 `npm run dev` 命令，开始开发

11. 执行 `npm run build` 命令，打包项目

    > 项目打包后是静态的网页，为了方便查看效果，可以使用 browser-sync 启动一个 web 服务查看:
    >
    > * npm install -g browser-sync
    > * 进入 dist 目录
    > * browser-sync start --server

## 新建 component

1. 在 **src/components** 中创建文件 **Foo.vue**

   ```html
   <template id="">
       <div class="foo">
           <span>{{msg}}</span>
       </div>
   </template>

   <script>
       export default {
           data() {
               // name: 'foo', // name 可以不要
               return {
                   msg: 'in component Foo',
               };
           },
       };
   </script>

   <style lang="scss">
       .foo {
           span {
               color: darkred;
           }
       }
   </style>
   ```

2. 在 **src/router/index.js** 中注册 component

   ```js
   import Vue from 'vue';
   import Router from 'vue-router';

   import Hello from '@/components/Hello';
   import Foo from '@/components/Foo';

   Vue.use(Router);

   export default new Router({
       routes: [{
           path: '/hello',
           component: Hello,
       }, {
           path: '/foo',
           component: Foo,
       }],
   });
   ```

3. 在 **src/App.vue** 中使用 component

   ```html
   <template>
       <div id="app">
           <img src="./assets/logo.png">

           <div>
               <ul class="menus">
                   <li class="menu-item"><router-link to="/">Home</router-link></li>
                   <li class="menu-item"><router-link to="/hello">Hello</router-link></li>
                   <li class="menu-item"><router-link to="/foo">Foo</router-link></li>
               </ul>
           </div>

           <div class="main">
               <router-view></router-view>
           </div>
       </div>
   </template>

   <script>
       export default {
           name: 'app',
       };
   </script>

   <style lang="scss">
       #app {
           font-family: '微软雅黑', Helvetica, Arial, sans-serif;
           -webkit-font-smoothing: antialiased;
           -moz-osx-font-smoothing: grayscale;
           text-align: center;
           color: #2c3e50;
           margin-top: 60px;
       }

       .menus {
           height: 20px;
           list-style: none;
           
           .menu-item {
               float: left;

               :not(first) {
                   margin-left: 10px;
               }

               :hover {
                   color: black;
               }
           }
       }
   </style>
   ```

   > `<router-view></router-view>`  就是被 component 替换的地方

## 引用静态文件

* 不能在 .vue 文件的模版中引用，需要在 index.html 中引用


* 使用 CDN 的 JS

  在 index.html 中使用 script 引用:

  ```html
  <body>
      <div id="app"></div>
      <!-- built files will be auto injected -->

      <script src="http://cdn.staticfile.org/layer/2.3/layer.js"></script>
  </body>
  ```

* 引用本地的 JS，不想被打包压缩

  先将其放入 static 目录，然后在 index.html 中使用 script 引用:

  ```html
  <body>
      <div id="app"></div>
      <!-- built files will be auto injected -->

      <script src="/static/lib/jquery.min.js" charset="utf-8"></script>
  </body>
  ```

## 参考资料

* [基于 vue-cli 快速构建](http://www.jianshu.com/p/2769efeaa10a)

