---
title: Atom 常用插件和快捷键
date: 2016-06-18 09:43:00
tags: [Mac, FE]
---

Atom 以前很慢，所以一直不想用，在 1.0 版本后启动差不多需要 1.5 秒，已经快了很多，尝试了下，感觉很好，插件更好用，例如格式化插件 atom-beautify，jshint 等、界面更舒服，现在已经从 SublimeText 替换到 Atom 了，以下为常用的几个插件

<!--more-->

## 常用插件

| 插件                           | 说明                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| atom-ternjs                    | 这是 Tern 项目的 Atom 插件，提供了了比较精确的代码补全功能，不止是匹配输入过的关键字，还可以提示 ECMAScript、DOM/BOM、NodeJS 的方法和属性，也能自动分析依赖的模块，给出补全提示，这样Atom 用起来就有点 IDE 的感觉了。 |
|linter-jshint|JavaScript 代码实时错误提示|
| highlight-selected             | 高亮所有和当前选中单词一样的单词，IDE 标配。                 |
| quick-highlight                | 和 `highlight-selected` 相似，但是能高亮多个选择的词         |
| pigments                       | CSS 颜色可视化，例如显示 rgba(0,0,0,0.8) 对应的颜色          |
| atom-beautify                  | 格式化插件, 支持很多语言，HTML，JS，CSS 等等<br>如果从包管理器里安装不了，就试试从命令行里 `apm install atom-beautify`<br>使用: `cmd + shift + p`, 输入 `beatify`，或者使用快捷键 `ctrl + alt + b` |
| atom-icons                     | 给文件加上图标                                               |
| jquery-snippets                | jQuery API 代码片段                                          |
| set-syntax                     | 方便的使用 `Command Palette` (cmd + shift + p 打开) 修改当前文件的语法 |
| atom-html-preview              | 在 Atom 里实时的预览 HTML 的内容，`Command Palette` 搜索 `preview`，或者使用快捷键 `ctrl + shift + h` |
| regex-railroad-diagram         | 正则表达式可视化: 鼠标放到正则表达式上面自动可视化，例如在 JS 文件里 `/\d+/g` |
| atom-no-tab-close-button       | 隐藏关闭按钮，避免误操作关闭 tab，使用 `cmd + w` 来关闭 tab  |
| docblockr                      | 函数名前输入 `/**` 按下回车生成文档的模版                    |
| tree-view-open-files           | Show open files in a list above the tree view.               |
| less-autocompile               | 保存的时候把 Less 自动编译为 CSS                             |
| sublime-style-column-selection | 列编辑                                                       |
| symbols-view-plus              | Atom 自带的 symbols-view 的增强版                            |
| symbols-navigator              | 函数显示上比 symbols-view-plus 更清晰                        |
| atom-history                   | 打开文件的记录                                               |
| atom-ide-ui                    | 这将在你的 Atom 中呈现 IDE 界面，但是要成为一个完全可工作的 IDE ，你还需要安装你的语言服务器支持。可以选择：ide-html、ide-typescript(TypeScript & JavaScript)、 ide-php、 ide-java、 ide-csharp |
| split-diff                     | 文件比较工具、合并工具                                       |
| git-time-machine               | 可以和 Git 仓库里任意版本进行比较，快捷键为 `Alt+T`          |
| git-log                        | 显示提交记录                                                 |
|hey-pane|Split 后可以按下 `CMD + Shift + K` 使得放大当前的 pane|
| OOOOOOOOOOOOOOOO               | OOO                                                          |

## 快捷键

| 功能             | 快捷键                   |
| -------------- | --------------------- |
| 显示所有函数         | `cmd + r`             |
| 显示隐藏 tree-view | `cmd + \`             |
| 拆分窗口           | 先按下 `cmd + k`，然后按下方向键 |

## 自定义快捷键

如果已经被使用了，先解绑，例如 `cmd + l`，然后再绑定新的快捷键

```js
'atom-text-editor':
    'cmd-l': 'unset!'
'.platform-darwin, .platform-win32, .platform-linux':
    'cmd-l': 'go-to-line:toggle'
    'cmd-o': 'outline-view:toggle'

'atom-text-editor':
    'cmd-d': 'unset!'
'atom-text-editor:not([mini])':
    'cmd-d': 'editor:delete-line'
'atom-text-editor[data-grammar~=html]':
    'alt-p': 'atom-html-preview:toggle'
```

## 自定义文件语法高亮规则

比如微信小程序的样式文件扩展名是 wxss，在 Atom 中打开默认是纯文本的样式，没有使用 CSS 的语法高亮，可以渲染它的修改语法规则为 CSS，但是只是一次生效的，下次打开工程又无效了。

可以手动的修改 config.cson(Atom -> Config)，在 core 下面创建 customFileTypes 来配置文件的打开类型:

```properties
core:
  customFileTypes:
    "source.css": [
      "wxss"
    ]
    "text.html.basic": [
      "wxml"
    ]
```

### 怎么查找 key，例如 text.html.basic 呢？

在 Packages 里搜索 **language-html**，点击打开，可以看到 `Scope: text.html.basic`，也即是在 Package 里搜索 **language- + 文件类型**，然后找到 **Scope** 即可。

## Tree View 隐藏文件

**config.cson**(Atom -> Config) 中编辑 **core -> ignoredNames**，添加需要隐藏的文件类型，**.*** 表示以点开头的所有文件:

```
core:
  ignoredNames: [
    ".git"
    ".hg"
    ".svn"
    ".DS_Store"
    "._*"
    "Thumbs.db"
    ".dll"
    ".vcxproj"
    ".suo"
    ".filters"
    ".user"
    ".sdf"
    ".*"
  ]
```

## 自定义 CSS

例如修改 **highlight-selected** 选中文本的背景色:

1. **Atom > Stylesheets...**

2. ```css
   atom-text-editor .highlights .highlight-selected.background .region {
       background-color: #3E4450;
       border: 0 solid red;
       border-radius: 0;
   }
   ```


## 去掉 ES6 的警告提示

I am using Atom's `linter`, `react`, and `linter-jshint`/`linter-jsxhint`. In my js files, I keep getting the warning

> Warning: 'import' is only available in ES6 (use 'esvertion: 6'). (W119)

First possibility, **recommended** : you can create a `.jshintrc` in you home directory and jshint will read it in case there is none in the project directory. You might need to restart Atom after.

`.jshintrc` 的内容:

```js
{
    "esversion" : 6
}
```

## 书签

当文档比较大时，跳转不方便，可以使用书签来进行跳转：

* 按下 `CMD + F2` 给当前行添加书签
* 按下 `CTL + F2` 显示所有书签，点击书签就会跳转到书签所在行