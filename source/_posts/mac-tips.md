---
title: Mac Tips
date: 2016-04-08 00:16:18
tags: Mac
---

Mac 使用中的一些常用操作和命令。

<!--more-->

## Vim 基础

下面列出一些 Vi 常用的命令和设置:

* 搜索高亮: `:set hlsearch`
* 显示行号: `:set number`
* 可使用鼠标点击: `:set mouse=a`
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
* 光标移到屏幕顶部: `H`
* 光标移到屏幕中部: `M`
* 光标移到屏幕底部: `L`
* 单词出现的次数: `:%s/pattern//gn`

Vi 内全局替换

```
# 使用 str2 替换 str1
:%s/str1/str2/g

# 多个匹配，例如把所有的 '\t' 替换为 ','
# 一个或多个用 '\+' 而不是 '+'
:%s/\t\+/,/g
```

如果要长期有效，可以在 `~/.vimrc` 里加上对应的设置，例如

```
set hlsearch
set number
colorscheme torte
set mouse=a
syntax on
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

## 查找后打开文件

```
find . -name mac-tips.md | xargs open
```

## 统计多个文件的行数

```
find . -name "*.java" | xargs wc -l
```

## awk 输出列

```
// 输出第一列
echo 1 2 3 4 5 | awk '{ print $1 }'

// 输出第三列到最后一列
echo 1 2 3 4 5 | awk '{ for (i=3; i<=NF; i++) print $i }'

// 列求和
cat m.txt | awk '{a+=$1}END{print a}'
```

## sort & uniq
```
// 按行排序，并去掉重复行
sort | uniq

// 按行排序，并去掉重复行，统计最后的行数
sort | uniq -c
```

## zip

```
// 把文件夹 H5 和文件 x.html 压缩成 result.zip
zip -r result.zip H5 x.html
```

## unzip

```
// -d 指定输出目录
// 如果没有 -d + targetDir，则解压到当前目录
unzip signup.zip [-d targetDir]
```

## tar

```
// 压缩为 archive.tar
tar -cf archive.tar file1 file2 file3

// 解压 archive.tar
tar -xf archive.tar

# Create compressed gzip archive.
tar -czf file.tar.gz file1 file2

# Extract .gz archive.
tar -xzf file.tar.gz
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

## 计算文件的 MD5
```
md5 fileName
```

## 计算字符串 MD5

```
# 下面 3 种方式都可以
md5 -s Welcome
md5 -s 'Welcome'
md5 -s "Welcome"

echo -n Welcome | md5
echo -n 'Welcome' | md5
echo -n "Welcome" | md5
```

## 输出文件的十六进制

```
hexdump -C Main.java | head -n 20
```

## 重新加载声卡驱动

合上盖子后再打开有时候可能会没声，注销再登陆后就有声音了，或者用命令重新加载声卡驱动也可以解决

```
sudo kextunload /System/Library/Extensions/AppleHDA.kext
sudo kextload /System/Library/Extensions/AppleHDA.kext
```

## 递归创建目录

```
# p 是 plus 之意
mkdir -p a/b/c
```

## Chrome 清楚缓存插件

Chrome 应用商店里搜索 **clear Cache**，找到 `clear Cache, clean cache, 清理缓存`并安装

## 查看文件夹大小

```
du -sh
```

## cp  复制文件夹

```
# 把文件夹 srcDir 复制到 destDir, srcDir 为 destDir 的子文件夹
cp -r srcDir destDir

# 把文件夹 srcDir 下的文件及子文件夹复制到 destDir
cp -r srcDir/ destDir
```

## scp 上传和下载文件

* 上传文件: `scp /Users/Biao/Desktop/a.zip root@192.168.82.130:/root`
* 下载文件: `scp root@192.168.82.130:/root/a.zip /Users/Biao/Desktop`

## mv 移动目录

```
# 移动目录和重命名一样
mv srcDir destDir/
```

## Quick Look plugins

可以使用 **brew cask** 安装 Quick Look 插件，具体请参考 <https://github.com/sindresorhus/quick-look-plugins>。

## Safari 插件

* [油猴 Tampermonkey](/download/tampermonkey.safariextz.zip)

  脚本下载地址: <https://greasyfork.org/zh-CN/scripts/>

  例如安装优酷的 HTML5 播放器插件，可以搜索 **优酷**

## 列出 Node 安装的软件

```
ls `npm root -g`
```

## 删除所有 Vmware 的进程

```
kill -9 `ps aux | grep -i vmware | grep -v grep | awk '{print $2}'`
```

## 去掉截图阴影

```
defaults write com.apple.screencapture disable-shadow -bool true ; killall SystemUIServer
```

## 更新文件图标

```
sudo rm -rfv /Library/Caches/com.apple.iconservices.store; sudo find /private/var/folders/ \( -name com.apple.dock.iconcache -or -name com.apple.iconservices \) -exec rm -rfv {} \; ; sleep 3;sudo touch /Applications/* ; killall Dock; killall Finder
```

## 隐藏显示文件

```
alias hide="chflags hidden *"
alias show="chflags nohidden *"
```

## 查看端口占用

使用 `lsof -i:port` 查看端口是否被某个进程占用了，例如 `lsof -i:8080`。

## 显示后台进程

使用 `jobs -l` 查看当前终端(标签页)启动的后台进程。关闭终端后，在另一个终端 jobs 无法看到后台跑得程序了，此时可利用 `ps` 命令。

## .bash_profile

```bash
export GRADLE_HOME="/usr/local/gradle"
export MYSQL_HOME="/usr/local/Cellar/mysql/5.7.13"
export NGINX_HOME="/usr/local/Cellar/openresty/1.11.2.5/nginx/sbin"
export PATH="$PATH:$GRADLE_HOME/bin:$MYSQL_HOME/bin:$NGINX_HOME"
export JAVA_HOME=`/usr/libexec/java_home -v 1.8.0_77`
# export JAVA_HOME=`/usr/libexec/java_home -v 9`

alias ll="ls -lh"
alias lla="ls -lha"
alias tcpCount="netstat -n | awk '/^tcp/ {++state[\$NF]} END {for(key in state) print key,\"\t\",state[key]}'"
alias rm="rmtrash"

# Hexo
alias hc="hexo clean"
alias hg="hexo generate"
alias hs="hexo server -g"
alias hd="hexo clean; hexo generate; hexo deploy"
alias gs="cd /Users/Biao/GitBook/Library/xtuer/notes; gitbook serve ."
alias notes="cd /Users/Biao/GitBook/Library/xtuer/notes; gitbook serve ."

alias cddesk="cd ~/Desktop"
alias cdblog="cd /Users/Biao/Documents/workspace/Blog"
alias cdjava="cd /Users/Biao/Documents/workspace/Java"
alias cdnotes="cd /Users/Biao/GitBook/Library/xtuer/notes"
alias cdtemp="cd /Users/Biao/Temp"
alias ngrokStart="/Applications/ngrok clientid 497cb09d080c5594"
alias cdbrew="cd /usr/local/Cellar"
alias hide="chflags hidden *"
alias show="chflags nohidden *"
alias cal="cal 2018"

# Theme
export CLICOLOR=1
export LSCOLORS=gxfxaxdxcxegedabagacad
export LC_CTYPE=en_US.UTF-8
export LC_ALL=en_US.UTF-8

function powerline_shell() {
    export PS1="$(/Applications/powerline-shell/powerline-shell.py $? 2> /dev/null)"
}
export PROMPT_COMMAND="powerline_shell; $PROMPT_COMMAND"
```

