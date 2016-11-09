---
title: Hexo 环境搭建
date: 2016-04-03 17:49:00
tags: Hexo
---

Welcome to [Hexo](https://hexo.io/)!   
This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

<!--more-->

## Install Node
Hexo 需要 Node

* Mac 安装 Node，可以使用 Homebrew 安装: `brew install node`
* Windows 安装 Node，进入 <https://nodejs.org/en/> 下载安装

> Node 就是 Node.js

## 使用 Node 的淘宝镜像
由于网络的问题，访问 Node 的默认仓库有可能会有问题，很多东西都下载不下来，所以可以使用淘宝的 Node 的镜像，命令行里执行 

```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```
即可，具体可参考 <http://npm.taobao.org>

## Install Github Client
Hexo 和 Github 一起使用就可以搭建一个免费的博客网站

* Mac 安装 Github 客户端，进入 <https://desktop.github.com> 下载客户端
* Windows 安装 Github 客户端，如果在线安装的话，由于网络的问题绝大多数时候都会安装失败，所以可以使用其他人提供的离线安装包，知乎这个帖子有好多个可选择 <https://www.zhihu.com/question/23110947>

> 提示: 安装 Github 客户端后，下面的操作最好是在 `Git Shell` 中进行，因为后面要用到 Git 的命令。Windows 中 安装 GitHub 客户端后会在桌面会创建一个 `Git Shell` 的快捷方式，或者 `开始菜单 > 所有程序 > GitHub, Inc > Git Shell`

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
* 申请 Github 账号，例如名字为 `xtuer`
* 在 Github 创建一个名字为 `xtuer.github.io` 的仓库
* 我们博客的网站自动为 <http://xtuer.github.io>
* 安装 hexo 的 git 插件

    ```
    $ npm install hexo-deployer-git --save
    ```

* 在 `Blog/_config.yml` 中配置 git

    ```
    deploy:
      type: git
      repo: git@github.com:xtuer/xtuer.github.io.git
    ```

* 发布时需要执行下面三步

    ```
    $ hexo clean
    $ hexo generate
    $ hexo deploy
    ```

    > 注意，有时候发布时会提示你没有权限访问 Github 的仓库，那是因为 ssh 访问需要的验证文件无效了，需要更新一下，最简单的就是用 Github 的客户端先访问一下，然后再发布就可以了
* 使用上面的命令发布好博客后，访问 <http://xtuer.github.io>，可以看到我们创建的博客能从网络上访问了

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
