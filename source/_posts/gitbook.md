---
title: GitBook 入门
date: 2017-08-20 17:05:58
tags: Hexo
---

**GitBook** 使用 Markdown 来写文档，官网已经有 5 万多本使用 GitBook 写的书了。现在不少公司都开始用 GitBook 来写项目文档、使用手册等。下面就简要的介绍怎么使用 GitBook:

1. 安装 Git
2. 安装 Nodejs
3. 安装 GitBook: `npm install gitbook -g`
4. 安装 GitBook-Cli: `npm install -g gitbook-cli`
5. 下载 GitBook Editor, GitBook 官方提供的编辑器: <https://www.gitbook.com/editor><!--more-->

## 编辑

需要的环境已经准备好了，打开 GitBook Editor 就可以开始写了，左边是章节目录，右边是编辑区：

![](/img/normal/gitbook-editor.png)

编辑后点击保存，然后同步到 GitBook 的仓库。

## 浏览器中查看

给其他人演示时一般都是用浏览器，而不是打开 GitBook Editor。进入项目的根目录，就是有 **README.md** 和 **SUMMARY.md** 的那个目录

1. 执行命令 `gitbook serve .`
2. 浏览器里访问 <http://localhost:4000> 

## 插件

Gitbook 本身功能丰富，但同时可以使用插件来进行个性化定制。[Gitbook 插件](https://plugins.gitbook.com/browse) 里已经有100多个插件，可以在 `book.json` 文件的 `plugins` 和 `pluginsConfig` 字段添加插件及相关配置，添加后别忘了进行安装。

```json
{
    "plugins": ["expandable-chapters", "prism", "-highlight", "navigator"],
    "pluginsConfig": {
        "prism": {
            "css": [
                "prismjs/themes/prism-tomorrow.css"
            ]
        }
    }
}
```

* [Expandable chapters](https://github.com/DomainDrivenArchitecture/gitbook-plugin-expandable-chapters)

  生成 HTML 的章节不能展开和收缩，太多时不方便查看

* 语法高亮插件 [Prism](https://github.com/gaearon/gitbook-plugin-prism)

可参考 <http://zhaoda.net/2015/11/09/gitbook-plugins/>

## 导出 PDF

GitBook 导出 PDF 依赖 **ebook-convert**, calibre 中包含了 ebook-convert：

1. 安装 **calibre**，到 https://calibre-ebook.com 下载

2. 把 **ebook-convert** 加入到系统环境变量

   * Windows: 添加到 PATH

   * Mac:

     ```
     ln -s /Applications/calibre.app/Contents/MacOS/ebook-convert /usr/local/bin
     或者
     export PATH=$PATH:/Applications/calibre.app/Contents/MacOS
     ```

   * Linux: 参考 Mac

3. 执行导出命令

   * 进入项目目录
   * 执行 `gitbook pdf`
   * 在项目目录中会生成 **book.pdf**


## 参考资料

[GitBook 的使用和常用插件](http://www.tuicool.com/articles/zee2ui)

