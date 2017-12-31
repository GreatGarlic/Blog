---
title: 数据库常用基础
date: 2017-12-31 09:47:31
tags: DB
---

## 列的操作

* 增加列: 基本形式 `ALTER TABLE 表名 ADD 列名 列数据类型 [约束][AFTER 插入位置]`

  ```sql
  ALTER TABLE demo ADD count INT DEFAULT 0 NOT NULL
  ```

* 修改列: 基本形式 `ALTER TABLE 表名 CHANGE 列名称 列新名称 新数据类型`，数据类型一样时不会丢失数据，也就是重命名了

  ```sql
  ALTER TABLE demo CHANGE count size INT
  ```

* 删除列: 基本形式 `ALTER TABLE 表名 DROP 列名称`

  ```sql
  ALTER TABLE demo DROP count
  ```

## 查表更新

使用一个表的数据更新另一个表可使用关联查询进行更新

```sql
UPDATE demo d 
JOIN (SELECT course_id AS id, weight FROM course) AS c 
ON   d.id=c.id 
SET  d.count=c.weight
```

<!--more-->

## 类型转换

使用 `CAST` 进行类型转换: 例如使用 long 作为主键，返回给浏览器时 JS 不支持 long，超过 16 位时会导致溢出，可以把 long 转为字符串类型给浏览器(Jackson、Fastjson 可以设置输出 long 为字符串，这样就不需要在 SQL 里处理了)

```sql
SELECT CAST(id AS CHAR) AS id, name FROM demo
```

## 切换 0 和 1

如果是 0 则设置为 1，否则设置为 0，对于切换 true 和 false 很有用，很像三元运算符

```sql
UPDATE demo SET is_marked=IF(is_marked=0, 1, 0)
```

## 创建表

```sql
# ------------------------------------------------------------
# 单题
# ------------------------------------------------------------
DROP TABLE IF EXISTS `question`;

CREATE TABLE `question` (
    id bigint(20) unsigned NOT NULL            COMMENT '题目的 ID',
    type varchar(8) DEFAULT NULL               COMMENT '题目的类型',
    content mediumtext                         COMMENT '题目的内容',
    analysis mediumtext                        COMMENT '题目的解析',
    answer text                                COMMENT '题目的答案',
    score int(11) DEFAULT NULL                 COMMENT '题目的分值',
    difficulty int(11) DEFAULT NULL            COMMENT '题目的难度',
    subject_code varchar(64) DEFAULT NULL      COMMENT '题目的科目编码',
    knowledge_point_code varchar(8) DEFAULT '' COMMENT '题目的知识点编码',
    knowledge_point_id bigint(20) DEFAULT 0    COMMENT '题目的知识点 ID',
    is_marked tinyint(4) DEFAULT 0             COMMENT '是否被标记过，0 为未标记，1 为已标记',

    created_time  datetime DEFAULT NULL        COMMENT '创建时间，手动更新，插入时可以用 now()',
    modified_time timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后更新时间，自动更新', 

    PRIMARY KEY (id)
) ENGINE=InnoDB;
```

> 建议表的编码使用数据库的编码，不用单独指定。

