---
title: Hexo 使用友言评论
date: 2017-03-25 20:23:34
tags: Hexo
---

在博客系统里增加评论功能，如果自己做的话，需要准备服务器端(PHP 和 Java 等实现)和数据库用于评论的提交、读取和保存。如果只是简单的使用评论功能，对评论的数据分析、安全性等要求不高，可以使用第三方提供的评论系统，例如 **友言**，集成起来也很简单，只要在页面的 HTML 代码里加入一小段代码即可。<!--more-->

## 注册友言账号

访问 <http://www.uyan.cc>，注册，然后会得到你的网络代码，例如

```html
<!-- UY BEGIN -->
<div id="uyan_frame"></div>
<script type="text/javascript" src="http://v2.uyan.cc/code/uyan.js?uid=232323"></script>
<!-- UY END -->
```

> 上面的 232323 根据每个人不同而不同。

## 使用友言评论

把上面友言的代码复制到主题的 **layout/_partial/article.ejs** 文件中，例如放在 `<section id="comments">` 下，我的如下

```html
<% if (!index && post.comments) { %>
    <!-- UY BEGIN -->
    <section id="comments">
        <div id="uyan_frame"></div>
        <script type="text/javascript" src="http://v2.uyan.cc/code/uyan.js?uid=232323"></script>
    </section>
    <!-- UY END -->
<% } %>
```

> **!index** 表示非主页中才显示评论的功能。

**注意**：友言评论在本地可以看到效果，但是不能发言，只有把博客发布到网上后才能发言。本地发言会提示:

```
抱歉，当前网页参数有误，操作失败
```

