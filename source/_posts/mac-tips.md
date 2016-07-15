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

## awk 输出列
```
// 输出第一列
echo 1 2 3 4 5 | awk '{ print $1 }'

// 输出第三列到最后一列
echo 1 2 3 4 5 | awk '{ for (i=3; i<=NF; i++) print $i }'
```

## sort & uniq
```
// 按行排序，并去掉重复行
sort | uniq

// 按行排序，并去掉重复行，统计最后的行数
sort | uniq -c
```

## unzip
```
// -d 指定输出目录
// 如果没有 -d + targetDir，则解压到当前目录
unzip signup.zip -d targetDir
```

## 查看不同状态的链接数量
```
netstat -n | awk '/^tcp/ {++state[$NF]} END {for(key in state) print key,"\t",state[key]}'

输出:
SYN_SENT    1
CLOSE_WAIT  4
ESTABLISHED 7
```

## 转换文件编码
```
// 单个文件转码
iconv -f GB18030 -t UTF8 201607-data.txt > 201607.txt

// 查找文件并转码
find *.txt -exec sh -c "iconv -f GB18030 -t UTF8 {} > {}.txt" \;
```

## 显示目录树
1. 使用 brew 安装 tree: `brew install tree`
2. 执行命令 `tree` 就能显示当前目录的树形结构

    ```
    .
    ├── LICENSE.txt
    ├── build.gradle
    └── src
        ├── main
        │   ├── java
        │   ├── resources
        │   └── webapp
        │       └── WEB-INF
        │           └── web.xml
        └── test
            ├── java
            └── resources
    
    9 directories, 3 files
    ```
