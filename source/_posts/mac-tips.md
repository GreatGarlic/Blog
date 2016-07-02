---
title: Mac Tips
date: 2016-07-02 16:16:18
tags: Mac
---

Mac 使用中的一些常用操作

<!--more-->

## Vim 基础
* 搜索高亮: `:set hlsearch`
* 显示行号: `:set number`
* 搜索: `/+keyword`
* 大小写不敏感搜索: 
    * `/+keyword\c`
    * `:set ic`，然后 `/+keyword`
* 删除当前行: `dd`
* 删除光标后的字符: `x`
* 复制当前行: `yy`
* 粘贴复制的行: `p`
* 回到第一行: `gg`
* 跳到最后行: `GG`
* 回到行首: `0`
* 跳到行尾: `shift + $`
* 光标处插入: `i`
* 光标后插入: `a`
* 插入新一行: `o`
* 撤销操作: `u`

如果要长期有效，可以在 `~/.vimrc` 里加上对应的设置，例如

```
set hlsearch
set number
```

## 查看使用端口的 PID
`lsof -n -P | grep :80`

## 使用文件名查找
`find dir -name filename`，例如 `find . -name mvc.xml`

## 使用文件大小查找
`find . -size +200M`

## 使用文件修改时间查找
`find . -ctime -10s`

## 查找后使用 grep
`find . -name "*.java" | xargs grep -n --color "topic"`
