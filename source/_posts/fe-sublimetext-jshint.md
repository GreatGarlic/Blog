---
title: Sublimetext 安装 jshint
date: 2016-06-04 10:35:06
tags: FE
---

## 一、安装 jshint 的依赖
1. 由于 `jshint` 是依赖 `Node.js` 的，所以要先安装 `Node.js`，安装了 `brew` 的同学可以直接在 Terminal 使用以下命令

    ```
    brew install node
    ```
2. 安装 `jshint`

    ```
    npm install -g jslint
    ```
3. 测试 `jshint`: 写个 js 文件，例如某些行不用分号结束，乱赋值等，然后用下面的命令测试，会输出不规范的提示

    ```
    jshint test.js
    ```

## 二、安装 SublimeLinter 及 jshint 插件
1. 安装 SublimeLinter
2. 安装 SublimeLinter-jshint
3. 安装成功后就可以到 Sublimetext 里测试了，右键 SublimeLinter->Show All Erroes，就会有提示了，也会有实时的错误提示

## 三、参考
* [sublime3在mac环境下安装linter](http://javenl.github.io/other/2015/03/05/sublime插件安装.html)
* [借助 SublimeLinter 编写高质量的 JavaScript & CSS 代码](http://www.cnblogs.com/lhb25/archive/2013/05/02/sublimelinter-for-js-css-coding.html)
