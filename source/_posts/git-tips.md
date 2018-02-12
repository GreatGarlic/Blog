---
title: Git Tips
date: 2016-07-10 12:01:44
tags: [Mac, Java, Util]
---

Git 的常用命令

<!--more-->

*   untracked - 新增的文件，Git 根本不知道它的存在
* not staged - 被索引过又被修改了的文件
* staged - 通过 git add 后被即将被提交的文件
* git reflog
* 版本回退: git reset --hard versionId
* 创建分支: git branch branchName
* 切换分支: git checkout branchName
* 创建并切换分支: git checkout -b branchName
* 使用 git fetch 和 git pull 都可以更新远程仓库的代码到本地，但是它们之间还是有区别
    * git fetch: 从远程获取最新的版本到本地的 tmp 分支上，之后再进行比较，决定是否合并

        ```
        git fetch origin master       # 从远程的 origin 仓库的 master 主分支更新最新的版本到origin/master分支上
        git diff master origin/master # 比较本地的master分支和origin/master分支的差别
        git merge origin/master       # 合并内容到本地 master 分支
        ```
    * git pull：从远程获取最新版本并 merge 到本地 (git fetch 更安全一些)

        ```
        git pull origin master        # 相当于 git fetch 和 git merge 的合并
        ```

---

![](/img/git/git-1.jpg)
![](/img/git/git-commands.png)
![](/img/git/git-commands.jpg)
![](/img/git/git-cheatsheet.png)
![](/img/git/git-flow.png)

## 搭建Git服务器

[搭建 Git 服务器](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/00137583770360579bc4b458f044ce7afed3df579123eca000)

> 初始化仓库用命令 `git init --bare sample.git` 而不是 `git init`

[使用 Gitolite 搭建 Git 服务器](http://blog.hwangjr.com/2016/01/14/使用Gitolite搭建Git服务器/)

> 不需要手动在 git 服务器中添加新用户或新仓库，gitolite 的用户，仓库和权限规则是使用一个名为 **gitolite-admin** 的特殊仓库进行维护，需通过修改该仓库并合并 push 到服务器中:
>
> * 创建新仓库只需要修改 `gitolite-admin/conf/gitolite.conf`，而不是用 git init --bare 命令
> * 添加新用户只需要把用户的 ssl 公钥如 **biao.pub** 放到 `gitolite-admin/keydir`，biao 是用户名，在 gitolite.conf 里用到

## 中文名乱码

例如使用 git status 输出 modified:   "template-web-gradle/doc/\344\275\277\347\224\250\350\257\264\346\230\216.md"，中文名的路径显示不正常，调用 `git config core.quotepath false` 后就可以了.

## 名字大小写不敏感

Git 是大小写不敏感的，导致跨操作系统共享的 Git 仓库就会遇到上面的情况。如果重命名的文件或文件夹只有大小写不同，那么对 Git 来说甚至都没有变化。下面介绍解决 Git 大小写不敏感导致的重命名无效的办法: **先将文件夹重命名为临时文件夹，然后再从临时文件夹恢复成正常文件夹。**

> 注意: 中间需要先 commit 一次，否则会存在两份文件夹！

下面以重命名 Default-CMD.png 为 default-cmd.png 为例:

```
$ git mv Default-CMD.pngs Default-CMD.bak.png
$ git add .
$ git commit -m "改名（第 1/2 步）"

$ git mv Default-CMD.bak.png default-cmd.png
$ git add .
$ git commit -m "改名（第 2/2 步）"

$ git push
```

## 参考资料

* [一入前端深似海，从此红尘是路人系列第十弹之如何合理利用Git进行团队协作 (一)](https://my.oschina.net/qiangdada/blog/800093)
* [一入前端深似海，从此红尘是路人系列第十一弹之如何合理利用Git进行团队协作 (二)](https://my.oschina.net/qiangdada/blog/808527)
* [解决 Git 重命名时遇到的大小写不敏感的问题](https://walterlv.oschina.io/post/case-insensitive-in-git-rename.html)


