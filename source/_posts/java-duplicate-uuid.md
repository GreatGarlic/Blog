---
title: 测试 Java 生成 UUID 是否重复
date: 2017-06-16 03:22:51
tags: Java
---

本文的目的是为了测试 Java 的 UUID.randomUUID() 生成 UUID 是否重复，使用了 2 种方式:

1. 多线程加 ConcurrentSkipListSet
2. 多线程加 MySQL
   1. 多线程生成 UUID
   2. 保存 UUID 到文件
   3. 导入文件中的 UUID 到 MySQL
   4. 使用 GROUP BY 和 HAVING 查找重复的 UUID

结果：尝试了多次，生成 1 万个，10 万，100 万个 UUID 都没有发现重复的情况。<!--more-->

## 方式一：多线程加 ConcurrentSkipListSet

```java
import java.util.UUID;
import java.util.concurrent.*;

public class Test {
    public static void main(String[] args) throws Exception {
        final int threadCount = 100; // 线程数量
        final int uuidCountPerThread = 1000; // 每个线程生成的 UUID 数量

        CountDownLatch threadLatch = new CountDownLatch(threadCount); // 用于等待所有线程启动完成

        ExecutorService executor = Executors.newCachedThreadPool();
        ConcurrentSkipListSet<String> uuids = new ConcurrentSkipListSet<String>();

        for (int i = 0; i < threadCount; ++i) {
            executor.submit(() -> {
                // 等待所有线程都运行到这里，然后都继续运行，差不多同时生成 uuid
                try { threadLatch.await(); } catch (InterruptedException e) { e.printStackTrace(); }

                for (int j = 0; j < uuidCountPerThread; ++j) {
                    String uuid = UUID.randomUUID().toString().replaceAll("-", "");
                    uuids.add(uuid);
                }
            });

            threadLatch.countDown();
        }

        executor.shutdown();

        Thread.sleep(2000); // 等待 UUID 生成完成，生成不同数量的 UUID 时需要调整
        System.out.println(uuids.size());
    }
}
```

实验结果：没有重复的 UUID。

## 方式二：多线程加 MySQL

### 生成 UUID

```java
import org.apache.commons.io.IOUtils;

import java.io.FileWriter;
import java.util.Collections;
import java.util.LinkedList;
import java.util.List;
import java.util.UUID;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Test {
    public static void main(String[] args) throws Exception {
        final int threadCount = 1000; // 线程数量
        final int uuidCountPerThread = 1000; // 每个线程生成的 UUID 数量

        CountDownLatch threadLatch = new CountDownLatch(threadCount);
        CountDownLatch allLatch = new CountDownLatch(threadCount * uuidCountPerThread);

        ExecutorService executor = Executors.newCachedThreadPool();
        List<String> uuids = Collections.synchronizedList(new LinkedList<String>());

        for (int i = 0; i < threadCount; ++i) {
            executor.submit(() -> {
                // 等待所有线程都运行到这里，然后都继续运行，差不多同时生成 uuid
                try { threadLatch.await(); } catch (InterruptedException e) { e.printStackTrace(); }

                for (int j = 0; j < uuidCountPerThread; ++j) {
                    String uuid = UUID.randomUUID().toString().replaceAll("-", "");
                    uuids.add(uuid);
                    allLatch.countDown();
                }
            });

            threadLatch.countDown();
        }

        executor.shutdown();

        allLatch.await(); // 等待 UUID 生成完成
        FileWriter writer = new FileWriter("/Users/Biao/Desktop/uuids.txt");

        // 保存 uuid 到文件
        for (String uuid : uuids) {
            IOUtils.write(uuid + "\n", writer);
        }

        IOUtils.closeQuietly(writer);
    }
}
```

### 导入文件中的 UUID 到 MySQL

创建数据库表 uuid:

```sql
CREATE TABLE `uuid` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `uuid` char(40) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=32822 DEFAULT CHARSET=utf8;
```

使用 CSV 的方式导入文件中的 UUID 到 MySQL:

```sql
LOAD DATA INFILE '/Users/Biao/Desktop/uuids.txt' 
INTO TABLE uuid 
FIELDS TERMINATED BY ',' 
OPTIONALLY ENCLOSED BY '"' 
ESCAPED BY '"' 
LINES TERMINATED BY '\n'
(uuid)
```

###  查找重复的 UUID

```sql
select uuid from uuid group by uuid having count(uuid)>1
```

实验结果：没有重复的 UUID。