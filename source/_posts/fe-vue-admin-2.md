---
title: Vue 后台管理简单框架（二）- 多页
date: 2017-06-01 18:38:44
tags: [FE, Vue]
---

[Vue 后台管理简单框架（一）](/fe-vue-admin-1)中介绍了单页 SPA 的实现，但是实际系统中后台管理的功能很可能是需要多页的，例如要开发一个学习系统，学生和老师的管理功能完全不一样，如果非要把它们放在一起使用 SPA 的方式也可以，左边菜单栏根据角色是老师或则学生来动态显示也是可以的，但是这样会导致管理页的代码很多，功能都放在一起，开发的时候可能不够清晰，增加开发难度，如果把它们分开，使用多页的方式来实现，功能模块就很清晰了，不失为一个好办法。还有例如 PC 的网页和移动设备的网页实现不同，如果放在同一个页面就需要做各种判断来确定对应设备显示的内容也会把很简单的逻辑搞的很复杂，使用不同的页面的话就会很清晰了。

下面就来介绍把 vue-cli 创建的工程改造为支持多页：

* 不同页面的文件放在不同的文件夹下

  > 每个页面都有自己的 router.js, index.js, index.vue, index.html

* 修改 4 个配置文件：
  * webpack.base.conf.js: 修改入口文件 entry
  * webpack.dev.conf.js: 修改 HtmlWebpackPlugin
  * webpack.prod.conf.js: 修改 HtmlWebpackPlugin，删除 CommonsChunkPlugin
  * .eslintrc.js <!--more-->

> 方法来源于 [基于 webpack2.x 的 vue2.x 的多页面站点](http://www.cnblogs.com/zqzjs/p/6834843.html)

## 文件结构

```
├── build
│   ├── ...
│   ├── webpack.base.conf.js
│   ├── webpack.dev.conf.js
│   └── webpack.prod.conf.js
├── src
│   ├── assets
│   ├── components
│   └── page
│       ├── student
│       │   ├── index.html
│       │   ├── index.js
│       │   ├── index.vue
│       │   ├── main.vue
│       │   └── router.js
│       └── teacher
│           ├── index.html
│           ├── index.js
│           ├── index.vue
│           ├── main.vue
│           └── router.js
├── static
```

> * 页面的文件放在 page 目录下，每个单页占用一个目录
> * 文件名字也修改下，这样看上去比较统一
>   * App.vue > index.vue
>   * Home.vue > main.vue
>   * main.js > index.js
>   * router/index.js > router.js
>   * 复制 index.html 到此目录下
> * index.js 中 `import App from './App'` 要修改为 `import App from './index.vue'`，注意 `.vue` 后缀

## 修改规则

* 执行 `npm install --save-dev eslint eslint-plugin-vue`

* 修改 **.eslintrc.js** 中的 extends 为

  ```js
  extends: ['plugin:vue/essential', 'airbnb-base', 'eslint:recommended', 'plugin:vue/recommended']
  ```

* 修改 **.eslintrc.js**，修改 rules，增加新的规则

  ```js
  'vue/html-self-closing': 0,
  'vue/html-indent': 0,
  'vue/max-attributes-per-line': 0,
  'vue/order-in-components': 0,
  'vue/html-end-tags': 0,
  'vue/no-parsing-error': 0
  ```

  详细信息请参考 <https://github.com/vuejs/eslint-plugin-vue>

## webpack.base.conf.js

```js
module.exports = {
    // 多页面的入口文件
    entry: {
        'page/student/index': './src/page/student/index.js',
        'page/teacher/index': './src/page/teacher/index.js',
    },
    ...
};
```

## webpack.dev.conf.js

```js
new HtmlWebpackPlugin({
    filename: './page/student/index.html',
    template: './src/page/student/index.html',
    chunks: ['page/student/index'],
    inject: true
}),
new HtmlWebpackPlugin({
    filename: './page/teacher/index.html',
    template: './src/page/teacher/index.html',
    chunks: ['page/teacher/index'],
    inject: true
}),
```

> 删除已有的 HtmlWebpackPlugin

## webpack.prod.conf.js

```js
new HtmlWebpackPlugin({
    filename:'./page/student/index.html',
    template:'./src/page/student/index.html',
    chunks:['page/student/index'],
    inject: true
}),
new HtmlWebpackPlugin({
    filename:'./page/teacher/index.html',
    template:'./src/page/teacher/index.html',
    chunks:['page/teacher/index'],
    inject: true
}),
```

> 删除已有的 HtmlWebpackPlugin 和 CommonsChunkPlugin

## 改进

上面实现多页的方式很直观、简洁，如果只有 4 个，5 个页面时这样修改配置没啥问题，但是有 20 个，30 个页面呢，配置就会写的很长，这个时候就可以使用函数来代替写硬代码的方式了。

webpack.base.conf.js:

```js
var glob = require('glob');

var entryJS = glob.sync('./src/page/**/*.js').reduce(function (prev, curr) {
    prev[curr.slice(6, -3)] = curr;
    return prev;
}, {});

module.exports = {
    entry: entryJS,
    ...
};
```

webpack.dev.conf.js 和 webpack.prod.conf.js:

```js
var glob = require('glob');

// item 为 ./src/page/student/index.html 和 ./src/page/teacher/index.html
var htmls = glob.sync('./src/page/**/*.html').map(function (item) {
    console.log(item);
    return new HtmlWebpackPlugin({
        filename: './' + item.slice(6),
        template: item,
        inject: true,
        chunks: [item.slice(6, -5)]
    });
});

module.exports = merge(baseWebpackConfig, {
    ...
    plugins: [
        ...
    ].concat(htmls)
});
```

## 测试

* 访问 <http://localhost:8888/page/student/index.html>
* 访问 <http://localhost:8888/page/teacher/index.html>

## 打包

`npm run build` 后生成的文件如下:

```
├── page
│   ├── student
│   │   └── index.html
│   └── teacher
│       └── index.html
└── static
    ├── css
    │   └── page
    │       ├── student
    │       │   ├── index.f9157a04ff70a02261a18caad5dc4058.css
    │       │   └── index.f9157a04ff70a02261a18caad5dc4058.css.map
    │       └── teacher
    │           ├── index.f9157a04ff70a02261a18caad5dc4058.css
    │           └── index.f9157a04ff70a02261a18caad5dc4058.css.map
    ├── fonts
    │   └── element-icons.b02bdc1.ttf
    ├── img
    │   └── avatar.jpg
    └── js
        ├── 0.74aa4aa0b2dfab165015.js
        ├── 0.74aa4aa0b2dfab165015.js.map
        ├── 1.3abf64768fdb6f210810.js
        ├── 1.3abf64768fdb6f210810.js.map
        ├── 2.150b5d962f5f32239c9c.js
        ├── 2.150b5d962f5f32239c9c.js.map
        ├── 3.4e5f52a424da88d1ad75.js
        ├── 3.4e5f52a424da88d1ad75.js.map
        ├── 4.a935cb93fcb87309c85e.js
        ├── 4.a935cb93fcb87309c85e.js.map
        ├── 5.a990e96495e161f6eaac.js
        ├── 5.a990e96495e161f6eaac.js.map
        ├── 6.88ce696032dd62445c9f.js
        ├── 6.88ce696032dd62445c9f.js.map
        ├── 7.b9b86098d990f9c09cf8.js
        ├── 7.b9b86098d990f9c09cf8.js.map
        └── page
            ├── student
            │   ├── index.4842fafe9a6a5f49d2d5.js
            │   ├── index.4842fafe9a6a5f49d2d5.js.map
            │   └── router
            │       ├── index.c5324c1ebb3b8cd6232d.js
            │       └── index.c5324c1ebb3b8cd6232d.js.map
            └── teacher
                ├── index.e1657eb4c71610d67cf4.js
                ├── index.e1657eb4c71610d67cf4.js.map
                └── router
                    ├── index.bd229432d6c462860b3e.js
                    └── index.bd229432d6c462860b3e.js.map
```

## 打包目录

`npm run build` 后生成的文件默认在 dist 目录中，在 Java Web 系统里很多时候是不行的，可以定制 build 的输出目录: 修改 **webpack.prod.conf.js** 中的 `output.path`

```js
output: {
    // path: config.build.assetsRoot,
    path: path.resolve(__dirname, '../../WEB-INF'),
    filename: utils.assetsPath('js/[name].[chunkhash].js'),
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
}
```

