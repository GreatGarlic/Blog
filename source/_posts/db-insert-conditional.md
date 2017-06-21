---
title: MySQL 基于条件判断的数据插入
date: 2017-06-15 16:35:40
tags: DB
---

在编写程序时，我们经常会遇到一些基于条件判断的逻辑，比如：判断该条数据是否已经在数据库中存在，如果不存在，则插入。

## 技巧一：使用 ignore 关键字

如果是用主键 primary 或者唯一索引 unique 区分了记录的唯一性，避免重复插入记录可以使用 **insert ignore into**。

当插入数据时，如出现错误时，如重复数据，将不返回错误，只以警告形式返回。所以使用 ignore 请确保语句本身没有问题，否则也会被忽略掉。<!--more-->

示例： 

```sql
INSERT IGNORE INTO book(name) VALUES('MySQL Manual')
```

## 技巧二：使用 replace into

REPLACE 的运行与 INSERT 很相像， 但是如果旧记录与新记录有相同的值，则在新记录被插入之前，旧记录被删除，即：尝试把新行插入到表中，当因为对于主键或唯一关键字出现重复关键字错误而造成插入失败时从表中删除含有重复关键字值的冲突行，再次尝试把新行插入到表中。

旧记录与新记录有相同的值的判断标准就是：表有一个 PRIMARY KEY 或 UNIQUE 索引，否则，使用一个 REPLACE 语句没有意义。该语句会与 INSERT 相同，因为没有索引被用于确定是否新行复制了其它的行。

语法：

```sql
REPLACE INTO table_name(col_name, ...) VALUES (...)
REPLACE INTO table_name(col_name, ...) SELECT ...
REPLACE INTO table_name SET col_name='value'
```

示例： 

```sql
REPLACE INTO book SELECT 1, 'MySQL Manual' FROM book
```

## 技巧三：ON DUPLICATE KEY UPDATE

当插入时主键冲突则执行后面的更新语句。

示例：

```sql
INSERT INTO book(name) VALUES('MySQL Manual') ON DUPLICATE KEY UPDATE name='Oracle'
```

## 技巧四：INSERT INTO IF EXISTS

根据 select 的条件判断是否插入，可以不光通过 primary 和 unique 来判断，也可通过其它条件。

示例： 

```sql
INSERT INTO book(name) 
SELECT 'MySQL Manual' FROM dual
WHERE NOT EXISTS(
    SELECT 1 FROM book WHERE id=23
)

# MyBatis 例子
<insert id="insertPaperKnowledgePointRelation" parameterType="KnowledgePoint">
    INSERT INTO paper_knowledge_point_relation(paper_id, knowledge_point_id)
    SELECT #{paperId}, #{knowledgePointId} FROM dual WHERE NOT EXISTS(
        SELECT 1 FROM paper_knowledge_point_relation
        WHERE paper_id=#{paperId} AND knowledge_point_id=#{knowledgePointId}
    )
</insert>
```

> 其中的 DUAL 是一个临时表，不需要物理创建。

[点击阅读原文](https://www.biaodianfu.com/mysql-insert-into-if-exists.html?utm_source=tuicool&utm_medium=referral)

