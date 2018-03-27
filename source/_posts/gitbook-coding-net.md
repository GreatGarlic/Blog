---
title: GitBook 使用 Coding.net 的 Pages 访问
date: 2018-03-27 00:44:36
tags: Hexo
---

GitBook 生成的静态网页文件在 `_book` 目录中，下面介绍怎么把它发布到 coding.net 的 Pages 服务中，这样就能够通过网络访问了。

1. 在 <https://coding.net> 创建一个账号 `xtuer`(下面请换为自己的账号)
2. 创建仓库 2 个仓库 fox 和 fox-doc (仓库名字随意取)：
   * fox-doc: GitBook 源文件
   * fox: GitBook 生成的静态文件
3. 克隆这 2 个仓库到本地的同一个文件夹下
   * `git clone git@git.coding.net:xtuer/fox.git`
   * `git clone git@git.coding.net:xtuer/fox-doc.git` <!--more-->
4. 命令行进入 fox-doc 目录: `cd ${fox-doc-path}`
   1. 增加 .gitignore 文件:

      ```
      _book/
      node_modules/
      Thumbs.db
      .DS_Store
      ```

   2. 在 fox-doc 中增加 GitBook 的文件 (可参考 http://qtdebug.com/gitbook)，主要是:
      * book.json
      * README.md
      * SUMMARY.md
   3. `gitbook install`
   4. `gitbook build` 生成静态网页到目录 `_book` 中

5. 命令行进入 fox 目录: `cd ../fox`

   1. 复制 `fox-doc/_book` 中的文件到 `fox` 目录下: `cp -r ../fox-doc/_book/* .`
   2. `git add -A`
   3. `git commit -m "O_O"`
   4. `git push`
   5. 访问 <https://coding.net> 中的仓库 fox，点击 `Pages 服务`，`部署来源` 选择 `master 分支`
   6. 访问 <http://xtuer.coding.me/fox> 就可以看到我们使用 GitBook 写的内容了

      > 格式为: `{user_name}.coding.me/{project_name}`，具体请参考 <https://coding.net/help/doc/pages/creating-pages.html>

## 脚本自动化

每次编辑和发布都要机械的重复上面的步骤，枯燥无味还容易出错，可以借助脚本自动化执行：

* 类 Unix 可以使用 shell 脚本 `fox-doc/deploy.sh` 进行自动化处理：

  ```
  gitbook build
  cd ../fox
  rm -rf *
  cp -r ../fox-doc/_book/* .
  git add -A
  git commit -m "O_O"
  git push
  ```

  在 fox-doc 目录中执行 `sh deploy.sh` 就可以把 GitBook 生成的网页发布到网上了


* Windows 可以使用 Bat 脚本 `fox-doc/deploy.bat` 进行自动化处理：

  ```
  call gitbook build
  cd ..\fox
  del /Q *
  rd /S /Q gitbook
  xcopy /E ..\fox-doc\_book\* .
  git add -A
  git commit -m "O_O"
  git push
  cd ..\fox-doc
  ```

  在 fox-doc 目录中执行 `deploy.bat` 就可以把 GitBook 生成的网页发布到网上了

  > `del /Q * 删除` 目录下的所有文件
  >
  > `rd /S /Q gitbook` 删除 gitbook 目录及其所有子目录和文件
  >
  > `call cmd` 在当前窗口运行第三方程序，运行完后继续往下执行
  >
  > `start cmd` 会打开新窗口运行第三方程序

---

提示：为了方便管理，`可以把 fox 目录放到 fox-doc 下面`，不过需要在 fox-doc 的 .gitignore 中加上规则 `fox/`，deploy.sh 对应的修改为

```
gitbook build
cd fox
rm -rf *
cp -r ../_book/* .
git add -A
git commit -m "O_O"
git push
```

还有一种办法是只使用一个仓库，master 分支管理 GitBook 的源文件，coding-pages 分支管理 GitBook 生成的网站文件并提供 Pages 服务，不过文件看上去比较乱，不如 2 个仓库的清晰。