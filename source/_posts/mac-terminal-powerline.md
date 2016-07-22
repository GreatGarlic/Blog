---
title: Mac Terminal Powerline-Shell
date: 2016-07-22 14:23:17
tags: Mac
---
如图给 Mac 的 Terminal 设置很酷的效果，还能显示 git 的不同状态

![](/img/mac/terminal-powershell.png)

<!--more-->

## 下載安裝 Powerline-Shell
1. 下载 Master 版: <https://github.com/banga/powerline-shell>
2. 解压进入 `powerline-shell` 的目录
3. 执行 `install.py`

    ```
    $ /path/to/powerline/install.py
    ```

    > Powerline-Shell 的文件不能删除，所以可以放到 `/Applications` 下

## 修改 Bash 的配置
1. `vi ~/.bash_profile`
2. 给 Bash 挂上 `powershell`

    ```
    function powerline_shell() {
        export PS1="$(~/powerline-shell/powerline-shell.py $? 2> /dev/null)"
    }
    export PROMPT_COMMAND="powerline_shell; $PROMPT_COMMAND"
    ```

    > 注意: `export PS1="$(~/powerline-shell/powerline-shell.py $? 2> /dev/null)"` 这里的路径必须是你系统上 Powerline-Shell 的路径

## 安装 Powerline 字体
1. 下载 Master 版: <https://github.com/powerline/fonts>
2. 安装字体:

    ```
    $ /path/to/powerline-fonts/install.sh
    ```

    > Powerline 字体安装完成后会自动复制到 `~/Library/Fonts`，所以安装完成后可以删除解压得到的文件

## 设置 Terminal 的字体
我选择了 `Sauce Code Powerline`，大小为 `13 pt`

![](/img/mac/terminal-font.png)

## 设置 Terminal 的背景
由于 Powerline 字体对半透明的支持不好，为了 Power Shell 的效果好一些，设置 Terminal 的背景为不透明

## 设置 CMD 的样式
下图在 Powershell 里叫 CMD，由于某些原因，显示效果有可能有问题

![](/img/mac/terminal-powershell-cmd-bad.png)

更理想的情况是小箭头和左边的背景颜色一样，如

![](/img/mac/terminal-powershell-cmd.png)

可以作如下修改

1. 修改 `/path/to/powerline/themes/default.py` 中的 `CMD_PASSED_BG` 的值，例如

    ```
    CMD_PASSED_BG = 231 #236
    CMD_PASSED_FG = 0
    CMD_FAILED_BG = 87 #161
    CMD_FAILED_FG = 0
    ```
2. 更新样式，执行

    ```
    $ /path/to/powerline/install.py
    ```
3. 继续步骤 1，直到修改到自己满意为止






