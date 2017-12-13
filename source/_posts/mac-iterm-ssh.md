---
title: iTerm ssh 自动登录
date: 2017-12-13 22:07:40
tags: Mac
---

使用 SSH 远程登录时:

1. 输入 `ssh root@host-ip`
2. 输入密码

每次都重复这样的操作，不仅麻烦，还要记忆好多东西，为了解决这个问题，借助 iTerm2 Profile 可以实现 SSH 自动登录:

1. 编写 expect 脚本
2. 使用此脚本创建 Profile
3. 使用此 Profile 打开新标签页

<!--more-->

## 编写 expect 脚本

```
#!/usr/bin/expect

set timeout 30
spawn ssh [lindex $argv 0]@[lindex $argv 1]
expect {
    "(yes/no)?"
    {send "yes\n";exp_continue}
    "password:"
    {send "[lindex $argv 2]\n"}
}
interact
```

把上面的脚本保存为 `/usr/local/bin/login.exp` 并为其加上可执行权限 `chmod +x /usr/local/bin/login.exp`，这个脚本接收 3 个参数: 用户名、远程主机 IP、密码。

## 使用此脚本创建 Profile

打开 iTerm2 的配置窗口(快捷键`CMD + ,`)，找到 Profiles，点击左下角的 `+` 创建一个 Profile，参考如下配置:

![](/img/mac/iterm-ssh-1.png)

> 小提示：如果设置了 iTerm2 启动时打开 `Hotkey Window`，那么每个设置为 Hotkey Window 的 Profile 都会对应的打开一个窗口，有多个 Profiles 一下会打开好多，可以在 Profile 的 Keys 中设置是否为 `Hotkey Window`。
>
> `Hotkey Window`: 使用快捷键可以打开和隐藏此 Profile 的窗口。

## 使用此 Profile 打开新标签页

使用配置好 SSH 的 Profile 打开新标签页时就进行自动登录了:

![](/img/mac/iterm-ssh-2.png)

## 参考资料

[如何在 iTerm2 中设置自动远程登录](http://m.blog.csdn.net/langsim/article/details/50445245)

