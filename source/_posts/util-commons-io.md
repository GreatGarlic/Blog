---
title: Commons IO 例子
date: 2016-05-23 14:19:24
tags: [Java, Util]
---

可以使用 Apache Commons-IO 操作文件:

说明 | 代码
---- | ---
获取文件名的后缀 | `FilenameUtils.getExtension(path)`
获取文件名不包含后缀 | `FilenameUtils.getBaseName(path)`
获取文件所在目录的路径 | `FilenameUtils.getFullPath(path)`
复制文件 | `FileUtils.copyFileToDirectory()`
复制文件夹 | `FileUtils.copyDirectory()`
移动文件 | `FileUtils.moveFileToDirectory()`
计算文件的 `check sum` | `FileUtils.checksumCRC32()`
递归的创建目录 | `FileUtils.forceMkdir(new File("/Users/Biao/Desktop/a/b/c"))`
还有更多文件相关的操作 | ……
