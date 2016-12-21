---
title: Git Tips
date: 2016-07-10 12:01:44
tags: [Mac, Java, Util]
---

Git 的常用命令

<!--more-->

*   untracked - 新增的文件，Git 根本不知道它的存在
*   not staged - 被索引过又被修改了的文件
*   staged - 通过 git add 后被即将被提交的文件
*   git reflog
*   版本回退: git reset --hard versionId
*   创建分支: git branch branchName
*   切换分支: git checkout branchName
*   创建并切换分支: git checkout -b branchName
*   使用 git fetch 和 git pull 都可以更新远程仓库的代码到本地，但是它们之间还是有区别
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

## 参考资料

* [一入前端深似海，从此红尘是路人系列第十弹之如何合理利用Git进行团队协作(一)](https://my.oschina.net/qiangdada/blog/800093)
* [一入前端深似海，从此红尘是路人系列第十一弹之如何合理利用Git进行团队协作(二)](https://my.oschina.net/qiangdada/blog/808527)

