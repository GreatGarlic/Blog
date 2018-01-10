---
title: Vue 后台管理简单框架（一）- 单页
date: 2017-05-30 16:03:34
tags: [FE, Vue]
---

[vue-cli 简单搭建项目框架](/fe-vue-skeleton/) 后我们知道了怎么使用 **vue-cli** 创建一个项目，但是页面比较简单，一般后台管理功能的界面会像下面这样子:

![](/img/fe/vue-admin-ui.png)

下面介绍怎么实现一个这样的界面，UI 的框架使用 [iView](https://www.iviewui.com/docs/guide/start-en)。<!--more-->

## 安装 iView

命令行进入前面创建的项目目录，安装 iView:

```js
npm install iview --save
npm install iview-loader --save-dev
```

## 引入 iView

* main.js: 在 main.js 中引入 iView

  ```js
  import iView from 'iview';
  import 'iview/dist/styles/iview.css';
  Vue.use(iView);
  ```

  请参考下面的 main.js:

  ```js
  import Vue from 'vue';
  import iView from 'iview';
  import 'iview/dist/styles/iview.css';

  import App from './App';
  import router from './router';

  Vue.use(iView);
  Vue.config.productionTip = false;

  new Vue({
      el: '#app',
      router: router,
      template: '<App/>',
      components: { App },
  });
  ```

* App.vue

  ```html
  <template>
      <div id="app">
          <router-view/>
      </div>
  </template>
  <script>
      export default {};
  </script>
  <style lang="scss">
      #app {
          font-family: '微软雅黑', Helvetica, Arial, sans-serif;
          -webkit-font-smoothing: antialiased;
          -moz-osx-font-smoothing: grayscale;
      }
  </style>
  ```

* Home.vue

  ```html
  <template>
      <div class="main">
          {{msg}}
          <Button type="primary" shape="circle" icon="ios-search" @click="click"></Button>
      </div>
  </template>

  <script>
      export default {
          data() {
              return {
                  msg: 'Search'
              };
          },
          methods: {
              click() {
                  alert(1);
              }
          }
      };
  </script>

  <style lang="sass">

  </style>
  ```

* router/index.js

  ```js
  import Vue from 'vue';
  import Router from 'vue-router';
  import Home from '@/components/Home';

  Vue.use(Router);

  export default new Router({
      routes: [{
          path: '/',
          name: 'Home',
          component: Home
      }, ],
  });
  ```

执行 `npm run dev` 启动项目，可以看到 iView 的按钮，点击也能够响应点击事件，iView 引入成功。

## 管理页面框架

到此能够使用 iView 了，开始截图的那个样子，可以自己单独写，也可以使用 [iView 的 Layout](https://www.iviewui.com/components/layout) 中提供的代码，修改 Home.vue 为下面的代码

```html
<template>
    <div class="layout">
        <Sider :style="{position: 'fixed', height: '100vh', left: 0, overflow: 'auto'}">
            <Menu active-name="1-2" theme="dark" width="auto" :open-names="['1', '2']">
                <Submenu name="1">
                    <template slot="title">
                        <Icon type="ios-navigate"/>订单中心
                    </template>
                    <MenuItem name="1-1">我的订单</MenuItem>
                    <MenuItem name="1-2">我的活动</MenuItem>
                    <MenuItem name="1-3">订单晒价</MenuItem>
                </Submenu>
                <Submenu name="2">
                    <template slot="title">
                        <Icon type="ios-keypad"/>关注中心
                    </template>
                    <MenuItem name="2-1">关注商品</MenuItem>
                    <MenuItem name="2-2">关注店铺</MenuItem>
                </Submenu>
                <Submenu name="3">
                    <template slot="title">
                        <Icon type="ios-analytics"/>资产中心
                    </template>
                    <MenuItem name="3-1">我的余额</MenuItem>
                    <MenuItem name="3-2">我的积分</MenuItem>
                </Submenu>
            </Menu>
        </Sider>

        <Layout :style="{marginLeft: '200px'}">
            <Header>
                <div class="user">
                    公孙二狗
                    <Avatar src="https://i.loli.net/2017/08/21/599a521472424.jpg" size="large"/>
                </div>
            </Header>
            <Content :style="{padding: '0 16px 16px'}">
                <Breadcrumb :style="{margin: '16px 0'}">
                    <BreadcrumbItem>Home</BreadcrumbItem>
                    <BreadcrumbItem>Components</BreadcrumbItem>
                    <BreadcrumbItem>Layout</BreadcrumbItem>
                </Breadcrumb>
                <Card dis-hover style="height: 600px">
                    Content
                </Card>
            </Content>
        </Layout>
    </div>
</template>

<script>
    export default {

    };
</script>

<style scoped lang="scss">
    .layout {
        background: #f5f7f9;
        position: relative;
        overflow: hidden;
    }

    .ivu-layout-header {
        padding: 0 16px;
        background: #fff;
        box-shadow: 0 2px 3px 2px rgba(0, 0, 0, .02);

        .user {
            float: right;
            font-weight: bold;
            font-size: 14px;
        }
    }
</style>
```

> active-name: 页面加载时高亮侧边栏中的 MenuItem 的 name
>
> open-names: 页面加载时要展开的 Submenu 的 name