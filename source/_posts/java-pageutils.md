---
title: 分页计算工具
date: 2017-05-04 09:07:35
tags: [Java, Util]
---

```java
package com.xtuer.util;

/**
 * 分页时需要计算某一页的起始位置，或则使用记录总数计算共有多少页，PageUtils 的任务就是计算分页时的数据:
 *     PageUtils.offset(pageNumber, pageSize) 用于计算起始位置
 *     PageUtils.pageCount(recordCount, pageSize) 用于计算共有多少页
 */
public class PageUtils {
    /**
     * 根据传入的页数、每页上的最多记录数计算这一页面的开始位置 offset
     *
     * @param pageNumber 页数
     * @param pageSize 每页上的最多记录数
     * @return 开始的位置 offset
     */
    public static int offset(int pageNumber, int pageSize) {
        // 校正参数，pageNumber 从 1 开始，pageSize 最小为 1
        pageNumber = Math.max(1, pageNumber);
        pageSize = Math.max(1, pageSize);

        int offset = (pageNumber -1) * pageSize; // 计算此页开始的位置 offset
        return offset;
    }

    /**
     * 根据传入的记录总数、每页上的最多记录数计算总页数 pageCount
     *
     * @param recordCount 记录的总数
     * @param pageSize 每页上的最多记录数
     * @return 总页数
     */
    public static int pageCount(int recordCount, int pageSize) {
        // 校正参数，recordCount 最小为 0，pageSize 最小为 1
        recordCount = Math.max(0, recordCount);
        pageSize = Math.max(1, pageSize);

        int page = (recordCount-1) / pageSize + 1;
        return page;
    }

    public static void main(String[] args) {
        System.out.println(offset(0, 10));
        System.out.println(offset(1, 10));
        System.out.println(offset(9, 10));
        System.out.println(offset(10, 10));

        System.out.println(pageCount(0, 10));
        System.out.println(pageCount(1, 10));
        System.out.println(pageCount(9, 10));
        System.out.println(pageCount(10, 10));
        System.out.println(pageCount(15, 10));
        System.out.println(pageCount(20, 10));
        System.out.println(pageCount(21, 10));
    }
}
```

