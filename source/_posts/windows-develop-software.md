---
title: Windows 开发者软件推荐
date: 2018-02-04 09:00:06
tags: Mac
---

## 包管理器 Chocolatey

Chocolatey 是一款专为 Windows 系统开发的、基于 NuGet 的包管理器工具，类似于 

* Node 的 npm
* MacOS 的 brew
* Ubuntu 的 apt-get
* CentOS 的 yml

Chocolatey 的设计目标是成为一个去中心化的框架，便于开发者按需快速安装应用程序和工具，官网为 <https://chocolatey.org>，安装很简单，根据说明安装即可。

### 常用命令

* 搜索: `choco search something`

* 列出: `choco list -lo`

* 安装: `choco install cmake`

  > 可访问 <https://chocolatey.org/packages> 查看已有的包和说明

* 卸载: `choco uninstall cmake`

* 升级: `choco upgrade cmake`

* 固定包的版本，防止包被升级: `choco pin windirstat` <!--more-->

## 微软官方神器第三方库管理工具 Vcpkg

Vcpkg 用来帮助我们在 Windows 方便地上获取第三方的 C/C++ 库，例如 curl, opencv 等，官网为 <https://github.com/Microsoft/vcpkg>，安装也是很简单，根据说明安装即可。

> Vcpkg 依赖 git 和 cmake，可以使用上面的 Chocolatey 安装 git 和 cmake: 
>
> * `choco install git`
>
>
> * `choco install cmake`，然后把 cmake 的 bin 目录添加到 PATH 环境变量里

### 常用命令

* 安装: `vcpkg install lib-name`
  * 安装 curl: `vcpkg install curl`
  * 安装 boost: `vcpkg install boost`
  * 安装 opencv: `vcpkg install opencv`
* 列出: `vcpkg list`
* 搜索:
  * `vckpg search`  列出所有可用的库，也可以在 ports 目录中看到
  * `vcpkg search curl` 列出搜索到的库
* 删除: `vcpkg remove curl`

### 补充说明

* 指定库的平台:

  默认安装的是 Windows x86 也就是 32 位的库，如果要安装 x64 的库，在库的名字后面加上 `:platform`，例如安装 x64 的 curl 的命令为 `vcpkg install curl:x64-windows`，具体还支持哪些平台，请访问官网的 `triplets`

* 库安装的位置: 

  * `${vcpkg}/installed/x86-windows`
  * `${vcpkg}/installed/x64-windows`

* 指定库的版本: 修改 `${vcpkg}/ports/package-name/profile.cmake` 中对应的信息，然后安装

* 可看看视频微软工程师对 Vcpkg 的介绍视频 <http://www.365yg.com/item/6499765776189227534>

## 命令行工具 ConEmu

CMD 的替代工具，**显示 UTF-8 字符时不会像 CMD 那样屏幕刷新不干净**，官网为 <https://github.com/Maximus5/ConEmu>。

ConEmu starts a console program in hidden console window and provides an alternative customizable GUI window with various features:

* smooth window resizing;
* tabs and splits (panes);
* easy run old DOS applications (games) in Windows 7 or 64bit OS (DosBox required);
* quake-style, normal, maximized and full screen window graphic modes;
* window font anti-aliasing: standard, clear type, disabled;
* window fonts: family, height, width, bold, italic, etc.;
* using normal/bold/italic fonts for different parts of console simultaneously;
* cursor: standard console (horizontal) or GUI (vertical);
* and more, and more...

## SSH 工具 FinalShell

FinalShell 是一体化的的服务器，网络管理软件，不仅是 ssh 客户端，还是功能强大的开发，运维工具，充分满足开发，运维需求。特色功能: 免费海外服务器远程桌面加速，ssh 加速，双边 tcp 加速，内网穿透，官网为 <http://www.hostbuf.com>。

**使用它是因为免费，并且 SSH 能够自动记住用户名密码，默认支持 UTF-8。**

