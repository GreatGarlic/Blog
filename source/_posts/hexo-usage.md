---
title: Hexo 常用命令
date: 2016-04-03 17:49:00
tags: Hexo
---

Welcome to [Hexo](https://hexo.io/)!   
This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

<!--more-->

## Install Node and npm
Hexo 需要 Node

```
$ brew install node
```

## Install Hexo and initialize Pages
```
$ npm install hexo-cli -g
$ hexo init Blog
$ cd Blog
$ npm install
$ hexo server
```

## Create a new post
```
$ hexo new "My New Post"
```

## Run server

``` bash
$ hexo server
```

## Generate static files

``` bash
$ hexo generate
```

## Deploy to git
需要 git 插件

```
$ npm install hexo-deployer-git --save
```

在 `Blog/_config.yml` 中配置 git

```
deploy:
  type: git
  repo: git@github.com:xtuer/xtuer.github.io.git
```

发布时需要执行下面三步

```
$ hexo clean
$ hexo generate
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/deployment.html)

## Use tag
* One tag: `tags: Programming`
* Multi tags: `tags: [Programming, Java, Spring]`

## Use image
配置 `Blog/_config.yml`

```
post_asset_folder: true
```

在 `Blog/source` 下创建图片的目录，如 `img`，md 中引用图片

```
![](/img/post-asset.png)
```

## 主页显示摘要
在 md 中，摘要内容的后面跟上 `<!--more-->`，否则主页会显示文章的全部内容

## 用别名简化命令
```
alias hd='hexo clean; hexo generate; hexo deploy'
alias hs='hexo server -g'
```

* 本地预览用 `hs`
* 发布时使用 `hd`

## 参考
* [Hexo + Github, 搭建属于自己的博客](http://www.jianshu.com/p/465830080ea9)
* [Hexo 搭建 Github 静态博客](http://www.cnblogs.com/zhcncn/p/4097881.html)
* [Hexo 主题 Light 修改教程](http://www.jianshu.com/p/70343b7c2fd3)
* [Hexo 独立博客新玩法](http://ibruce.info/2013/11/22/hexo-your-blog/)
