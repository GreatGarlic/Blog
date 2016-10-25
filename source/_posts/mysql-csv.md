---
title: MySQL 导入导出 CSV
date: 2016-07-23 11:47:05
tags: DB
---

有时需要把 MySQL 的数据导出为 CSV 格式的文件便于分析和传输，有的时候需要把 CSV 格式的内容导入到 MySQL，MySQL 支持对 CSV 格式的文件导入和导出。

<!--more-->

## MySQL 导出 CSV
```sql
SELECT * FROM demo WHERE id>2 ORDER BY id
INTO OUTFILE '/Users/Biao/Desktop/a.csv'  
FIELDS TERMINATED BY ','  
OPTIONALLY ENCLOSED BY '"'   
LINES TERMINATED BY '\n';
```

## MySQL 导入 CSV
```sql
LOAD DATA INFILE '/Users/Biao/Desktop/a.csv' 
INTO TABLE demo 
FIELDS TERMINATED BY ',' 
OPTIONALLY ENCLOSED BY '"' 
ESCAPED BY '"' 
LINES TERMINATED BY '\n'; 
```

还可以在 `()` 部分指定导入的列:

```sql
LOAD DATA INFILE '/Users/Biao/Desktop/a.csv' 
INTO TABLE demo 
FIELDS TERMINATED BY ',' 
OPTIONALLY ENCLOSED BY '"' 
ESCAPED BY '"' 
LINES TERMINATED BY '\n'
(id, description)
```

