---
title: Hexo 跳过指定文件的渲染
date: 2017-09-07 07:12:30
tags: Hexo
---

Hexo 博客中所见文章都是经由渲染的静态网页，而静态网页的样式都直接由 Hexo 的主题控制，所以 Hexo 博客大部分都呈现出一种高度的统一化与规范化。不过 Hexo 提供了跳过渲染功能，使得我们可以直接在博客中放入自定义网页: 在 `_config.yml` 配置中配置 `skip_render`:

如果要跳过 source 文件夹下的`test.html`，可以这样配置：

```
skip_render: test.html

```

> 注意，千万不要加上个`/`写成`/test.html`，这里只能填相对于 source 文件夹的**相对路径**。

如果要忽略 source 下的 test 文件夹下所有文件，可以这样配置：

```
skip_render: test/*

```

如果要忽略 source 下的 test 文件夹下`.html`文件，可以这样配置：

```
skip_render: test/*.html

```

如果要忽略 source 下的 test 文件夹内所有文件包括子文件夹以及子文件夹内的文件，可以这样配置：

```
skip_render: test/**

```

如果要忽略多个路径的文件或目录，可以这样配置：

```
skip_render:
    - test.html
    - test/*
```

参考:

* [Hexo 博客中跳过渲染，创建自定义网页](http://www.jianshu.com/p/f89428fce8d5)
* [Hexo 跳过指定文件的渲染](http://e12e.com/2016/06/05/hexo跳过指定文件的渲染/)

