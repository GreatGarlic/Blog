---
title: Linux Crontab
date: 2016-04-28 13:43:24
tags: Mac
---

在 Linux 的日常管理中，有时需要定时删除指定的文件，可以使用 `crontab` 来完成。

<!--more-->

## 显示 crontab
```
contrab -l
```

## 编辑 crontab
```
crontab -e
```

## crontab 语法
```
# 每个小时的 20 分删除匹配 db*.data 的文件，例如 1 点 20，2 点 20， 3 点 20，……
20 * * * * rm -rf /Users/Biao/Desktop/temp/db*.data

# 每天晚上 1 点半删除文件 
30 1 * * * rm -rf /Users/Biao/Desktop/temp/db*.data
```

* 前五个以 4 个空格分隔开的值依次表示：分、时、日、月、周，如果取所有值就是打 `*` 号。

如果你想周期性的运行一个任务，crontab 也接受范围指定，比如说一天中的早 8 点到晚 6 点每 2 小时（将会在8,10,12,14,16,18执行）执行删除命令

```
* 8-18/2 * * * rm -rf /Users/Biao/Desktop/temp/db*.data
```

* 第一个字段是分钟，取值范围：0-59
* 第二个字段是小时，取值范围：0-23
* 第三个字段是一个月中的第几天，取值范围：1-31
* 第四个字段是一年中的第几个月，取值范围：1-12
* 最后一个字段是一个星期中的第几天，以星期天开始依次的取值为0～7，0、7都表示星期天

还可以指定执行命令的用户，例如用户 root 执行删除文件的命令

```
30 1 * * * root rm -rf /Users/Biao/Desktop/temp/db*.data
```

