---
title: MySQL 中 datetime 和 timestamp 的区别
date: 2016-05-04 09:34:46
tags: DB
---

最佳实践：每个表都应该有一列使用 `timestamp` 用于自动记录更新时间，这一列不需要在 Java 类中出现，如果需要记录创建时间，则使用 `datetime`。

<!--more-->

## datetime
1. 允许为空值，可以自定义值，系统不会自动修改其值
2. 不可以设定默认值，所以在不允许为空值的情况下，必须手动指定 `datetime` 字段的值才可以成功插入数据
3. 虽然不可以设定默认值，但是可以在指定 `datetime` 字段的值的时候使用 `now()` 变量来自动插入系统的当前时间

> 结论: `datetime 类型适合用来记录数据的原始的创建时间`，因为无论你怎么更改记录中其他字段的值，datetime 字段的值都不会改变，除非你手动更改它。

## timestamp
1. 允许为空值，但是不可以自定义值，所以为空值时没有任何意义
2. 默认值为 `CURRENT_TIMESTAMP()`，其实也就是当前的系统时间
3. `timestamp` 列不可以用程序设置值，只能由数据库自动去修改(手动修改也可以 =_=!!!)，所以在插入记录时不需要指定 `timestamp` 字段的名称和 `timestamp` 字段的值，你只需要在设计表的时候添加一个 `timestamp` 字段即可，插入后该字段的值会自动变为当前系统时间
4. 以后任何时间修改表中的记录时，对应记录的 `timestamp` 值会自动被更新为当前的系统时间
5. TIMESTAMP列创建后的格式是

    ```sql
    `a` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
    ```

> 结论: `timestamp 类型适合用来记录数据的最后修改时间`，因为只要你更改了记录中其他字段的值，timestamp字段的值都会被自动更新。

## 参考
* [SQL 中 datetime 和 timestamp 的区别](http://blog.csdn.net/luoyin22/article/details/9068885)
* [MySQL5 日期类型 DATETIME 和 TIMESTAMP 相关问题详解](http://lavasoft.blog.51cto.com/62575/280284/)
