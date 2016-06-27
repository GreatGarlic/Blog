---
title: Atom 常用插件和快捷键
date: 2016-06-18 09:43:00
tags: [Mac, FE]
---

Atom 以前很慢，所以一直不想用，在 1.0 版本后启动差不多需要 1.5 秒，已经快了很多，尝试了下，感觉很好，插件更好用，例如格式化插件 atom-beautify，jshint 等、界面更舒服，现在已经从 SublimeText 替换到 Atom 了，以下为常用的几个插件

<!--more-->

<style>
table td {
    min-width: 200px;
}
</style>

插件                    |         说明
---------------------- | ----------------------
atom-ternjs            | 这是 Tern 项目的 Atom 插件，提供了了比较精确的代码补全功能，不止是匹配输入过的关键字，还可以提示 ECMAScript、DOM/BOM、NodeJS 的方法和属性，也能自动分析依赖的模块，给出补全提示，这样Atom 用起来就有点 IDE 的感觉了。
highlight-selected     | 高亮所有和当前选中单词一样的单词，IDE 标配。
quick-highlight        | 和 `highlight-selected` 相似，但是能高亮多个选择的词
pigments               | CSS 颜色可视化，例如显示 rgba(0,0,0,0.8) 对应的颜色
jshint                 | JavaScript 代码实时错误提示
atom-beautify          | 格式化插件, 支持很多语言，HTML，JS，CSS 等等<br>如果从包管理器里安装不了，就试试从命令行里 `apm install atom-beautify`<br>使用: `cmd + shift + p`, 输入 `beatify`，或者使用快捷键 `ctrl + alt + b`
atom-icons             | 给文件加上图标
jquery-snippets        | jQuery API 代码片段
set-syntax             | 方便的使用 `Command Palette` (cmd + shift + p 打开) 修改当前文件的语法
browser-plus           | 在 Atom 里实时的预览 HTML 的内容，`Command Palette` 搜索 `browser open`，或者使用快捷键 `ctrl + alt + o`
regex-railroad-diagram | 正则表达式可视化: 鼠标放到正则表达式上面自动可视化，例如在 JS 文件里 `/\d+/g`
atom-no-tab-close-button | 隐藏关闭按钮，避免误操作关闭 tab，使用 `cmd + w` 来关闭 tab

功能              |        快捷键
----------------- | -----------------
显示所有函数        | `cmd + r`
显示隐藏 tree-view | `cmd + \`
拆分窗口           | 先按下 `cmd + k`，然后按下方向键

自定义快捷键

* 如果已经被使用了，先解绑，例如 `cmd + l`

    ```js
    'atom-text-editor':
        'cmd-l': 'unset!'
    ```
* 绑定新的快捷键

    ```js
    '.platform-darwin, .platform-win32, .platform-linux':
        'cmd-l': 'go-to-line:toggle'
    ```
