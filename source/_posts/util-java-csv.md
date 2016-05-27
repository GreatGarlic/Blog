---
title: Java 读写 CSV
date: 2016-04-17 20:02:27
tags: [Util, Java]
---

## Gradle 依赖
```
'org.apache.commons:commons-csv:1.2'
```

<!--more-->

## test.csv
CSV 文件的第一行可以是列名，也可以没有。

```
x,y,ex,ey
1.1,1.2,1.3,1.4
2.1,2.2,2.3,2.4
3.1,3.2,3.3,3.4
4.1,4.2,4.3,4.4
```

## CsvReader.java
* 读取带有列名的 CSV 使用 `CSVFormat.EXCEL.withHeader().parse(reader)`
* 读取不带列名的 CSV 使用 `CSVFormat.EXCEL.parse(reader)`

```java
import org.apache.commons.csv.CSVFormat;
import org.apache.commons.csv.CSVRecord;

import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.Reader;

public class CsvReader {
    public static void main(String[] args) throws Exception {
        InputStream in = CsvReader.class.getClassLoader().getResourceAsStream("test.csv");
        Reader reader = new InputStreamReader(in);
        Iterable<CSVRecord> records = CSVFormat.EXCEL.withHeader().parse(reader);

        for (CSVRecord record : records) {
            System.out.printf("[x: %s, y: %s]\n", record.get("x"), record.get("y"));
        }
    }
}
```

输出

```
[x: 1.1, y: 1.2]
[x: 2.1, y: 2.2]
[x: 3.1, y: 3.2]
[x: 4.1, y: 4.2]
```

## CsvWriter.java
```java
import org.apache.commons.csv.CSVFormat;
import org.apache.commons.csv.CSVPrinter;

import java.io.IOException;

public class CsvWriter {
    public static void main(String[] args) throws IOException {
        // Create the CSVFormat object with "\n" as a record delimiter
        StringBuilder result = new StringBuilder();
        CSVFormat csvFormat =  CSVFormat.DEFAULT.withRecordSeparator("\n");
        CSVPrinter csvPrinter = new CSVPrinter(result, csvFormat);

        csvPrinter.printRecord("ID", "username", "password");
        csvPrinter.printRecord("1", "Alice", "Passw0rd");
        csvPrinter.printRecord("2", "Bob", "123456");
        csvPrinter.printRecord("3", "John", "Aloha");
        csvPrinter.close();

        System.out.println(result);
    }
}
```

输出

```
ID,username,password
1,Alice,Passw0rd
2,Bob,123456
3,John,Aloha
```

## 参考
* [Write/Read CSV Files with Apache Commons CSV Example](https://examples.javacodegeeks.com/core-java/apache/commons/csv-commons/writeread-csv-files-with-apache-commons-csv-example/)
