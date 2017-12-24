---
title: SQL Server 导出 CSV 和 XML
date: 2017-12-20 20:47:35
tags: DB
---

## 导出 XML

```
bcp "select * from tableName FOR XML AUTO, ROOT('Root')" queryout C:/x.xml -S(local) -T -r -c
```

* `-w`: 使用 UTF-16 编码，小端
* `-c`: 使用 GBK 编码
* `-c -C6501`: 使用 UTF-8 编码，但是有的计算机上不支持
* `-T`: 本机使用 `-T` 表示可信连接，如果是访问其他机器使用 `-U user -P pwd` 输入用户名和密码

> 注意：
>
> * 如果不使用 `-r`，则导出的 XML 每行最多有 2033 个字符，会破坏 XML，用了 `-r` 后就没有换行符了，整个 XML 的内容在同一行。
> * 内容中的 & 等特殊字符不会被转义就直接放到属性值里了，此时用 XML 库解析会出错。

## 导出 CSV

```
sqlcmd -S localhost -d dbName -E -o "csvFile.csv" -Q "select * from tableName" -W -w 999 -s ","
```

* `-W`: remove trailing spaces from each individual field
* `-s","`: sets the column seperator to the comma (,) 
* `-w 999`: sets the row width to 999 chars(this will need to be as wide as the longest row or it will wrap to the next line)
* `-U`: username
* `-P`: password
* `-h-1`: removes column name headers from the result

> 注意：sqlcmd 导出为 CSV 文件时，如果列中有逗号，那么导出的 CSV 文件会被破坏，还没找到好办法。

参考 [How to export data as CSV format from SQL Server using sqlcmd?](https://stackoverflow.com/questions/425379/how-to-export-data-as-csv-format-from-sql-server-using-sqlcmd)

