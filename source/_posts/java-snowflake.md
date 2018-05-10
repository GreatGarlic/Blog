---
title: 分布式 ID 生成算法 Snowflake
date: 2017-10-26 13:42:00
tags: Java
---

分布式系统中，有一些需要使用全局唯一 ID 的场景，这种时候为了防止 ID 冲突可以使用 36 位的 UUID，但是 UUID 有一些缺点，首先他相对比较长，另外 UUID 一般是无序的字符串。

有些时候我们希望能使用简单一些的 ID，并且希望 ID 能够按照时间有序生成，为了解决这个问题，Twitter 发明了 SnowFlake 算法，不依赖第三方介质例如 Redis、数据库，本地生成程序生成分布式自增 ID，这个 ID 只能保证在工作组中的机器生成的 ID 唯一，不能像 UUID 那样保证时空唯一。

Snowflake 把时间戳、工作组 ID、工作机器 ID、自增序列号组合在一起，生成一个 64bits 的整数 ID，能够使用 70 年，每台机器每秒可产生约 25 万个 ID。

Snowflake 也有自己的缺点，虽然不同 workId 的机器生成的 ID 永远不会相同，但是同一台机器当把时间往后回拨后，生成的 ID 就会重复，所以需要保持时间是网络同步的。

Snowflake 生成的 ID 的 bit 结构如下:

![](/img/java/snowflake.png)<!--more-->

下面用 Java 实现 Snowflake，是现成安全的，调用 `nextId()` 生成 ID:

```java
package com.xtuer.service;

import org.apache.commons.lang3.StringUtils;

/**
 * 使用 Snowflake 算法生成的 64 位 long 类型的 ID，结构如下:<br>
 * 0 - 0000000000 0000000000 0000000000 0000000000 0 - 00000 - 00000 - 000000000000 <br>
 * 1) 01 位标识，由于 long 在 Java 中是有符号的，最高位是符号位，正数是 0，负数是 1，ID 一般使用正数，所以最高位是 0<br>
 * 2) 41 位时间截(毫秒级)，注意，41 位时间截不是存储当前时间的时间截，而是存储时间截的差值(当前时间 - 开始时间)得到的值，
 *       开始时间截，一般是业务开始的时间，由我们程序来指定，如 IdWorker 中的 START_TIMESTAMP 属性。
 *       41 位的时间截，可以使用 70 年: (2^41)/(1000*60*60*24*365) = 69.7 年<br>
 * 3) 10 位的数据机器位，可以部署在 1024 个节点，包括 5 位 datacenterId 和 5 位 workerId<br>
 * 4) 12 位序列，毫秒内的计数，12 位的计数顺序号支持每个节点每毫秒(同一机器，同一时间截)产生 4096 个 ID 序号<br>
 *
 * SnowFlake 的优点是，整体上按照时间自增排序，并且整个分布式系统内不会产生 ID 碰撞(由数据中心 ID 和机器 ID 作区分)，并且效率较高。
 * 最多支持 1024 台机器，每台机器每毫秒能够生成最多 4096 个 ID，整个集群理论上每秒可以生成 1024 * 1000 * 4096 = 42 亿个 ID。
 */
public class IdWorker {
    /** 开始时间截(2017-01-01)，单位毫秒 */
    private static final long START_TIMESTAMP = 1483228800000L;

    /** 机器 ID 所占的位数 */
    private static final long WORKER_ID_BITS = 5L;

    /** 数据标识 ID 所占的位数 */
    private static final long DATACENTER_ID_BITS = 5L;

    /** 支持的最大机器 ID，结果是 31: 0B11111 */
    private static final long MAX_WORKER_ID = -1L ^ (-1L << WORKER_ID_BITS);

    /** 支持的最大数据中心 ID，结果是 31: 0B11111 */
    private static final long MAX_DATACENTER_ID = -1L ^ (-1L << DATACENTER_ID_BITS);

    /** 序列在 ID 中占的位数 */
    private static final long SEQUENCE_BITS = 12L;

    /** 机器 ID 向左移 12 位 */
    private static final long WORKER_ID_SHIFT = SEQUENCE_BITS;

    /** 数据中心 ID 向左移 17 位(12+5) */
    private static final long DATACENTER_ID_SHIFT = SEQUENCE_BITS + WORKER_ID_BITS;

    /** 时间截向左移 22 位(5+5+12) */
    private static final long TIMESTAMP_LEFT_SHIFT = SEQUENCE_BITS + WORKER_ID_BITS + DATACENTER_ID_BITS;

    /** 生成序列的掩码，这里为 4095(0B111111111111=0xFFF=4095) */
    private static final long SEQUENCE_MASK = -1L ^ (-1L << SEQUENCE_BITS);

    /** 工作机器 ID(0~31) */
    private long workerId;

    /** 数据中心 ID(0~31) */
    private long datacenterId;

    /** 毫秒内序列(0~4095) */
    private long sequence = 0L;

    /** 上次生成 ID 的时间截 */
    private long lastTimestamp = -1L;

    /**
     * 创建 ID 生成器的方式一: 使用工作机器的序号，范围是 [0, 1023]，优点是方便给机器编号
     *
     * @param workerId 工作机器 ID
     */
    public IdWorker(long workerId) {
        long maxMachineId = (MAX_DATACENTER_ID +1) * (MAX_WORKER_ID +1) - 1; // 1023

        if (workerId < 0 || workerId > maxMachineId) {
            throw new IllegalArgumentException(String.format("Worker ID can't be greater than %d or less than 0", maxMachineId));
        }

        this.datacenterId = (workerId >> WORKER_ID_BITS) & MAX_DATACENTER_ID;
        this.workerId = workerId & MAX_WORKER_ID;
    }

    /**
     * 创建 ID 生成器的方式二: 使用工作机器 ID 和数据中心 ID，优点是方便分数据中心管理
     *
     * @param datacenterId 数据中心 ID (0~31)
     * @param workerId     工作机器 ID (0~31)
     */
    public IdWorker(long datacenterId, long workerId) {
        if (workerId > MAX_WORKER_ID || workerId < 0) {
            throw new IllegalArgumentException(String.format("Worker ID can't be greater than %d or less than 0", MAX_WORKER_ID));
        }
        if (datacenterId > MAX_DATACENTER_ID || datacenterId < 0) {
            throw new IllegalArgumentException(String.format("Datacenter ID can't be greater than %d or less than 0", MAX_DATACENTER_ID));
        }

        this.workerId = workerId;
        this.datacenterId = datacenterId;
    }

    /**
     * 获得下一个 ID(该方法是线程安全的)，同一机器同一时间可产生 4096 个 ID，70 年内不生成重复的 ID
     *
     * @return long 类型的 ID
     */
    public synchronized long nextId() {
        long timestamp = timeGen();

        // 如果当前时间小于上一次 ID 生成的时间戳，说明系统时钟回退过这个时候应当抛出异常
        if (timestamp < lastTimestamp) {
            throw new RuntimeException(String.format("Clock moved backwards. Refusing to generate id for %d milliseconds", lastTimestamp - timestamp));
        }

        // 如果是同一时间生成的，则进行毫秒内序列
        if (lastTimestamp == timestamp) {
            sequence = (sequence + 1) & SEQUENCE_MASK;

            // 毫秒内序列溢出
            if (sequence == 0) {
                // 阻塞到下一个毫秒，获得新的时间戳
                timestamp = tilNextMillis(lastTimestamp);
            }
        } else {
            sequence = 0L; // 时间戳改变，毫秒内序列重置
        }

        // 上次生成 ID 的时间截
        lastTimestamp = timestamp;

        // 移位并通过或运算拼到一起组成 64 位的 ID
        return ((timestamp - START_TIMESTAMP) << TIMESTAMP_LEFT_SHIFT)
                | (datacenterId << DATACENTER_ID_SHIFT)
                | (workerId << WORKER_ID_SHIFT)
                | sequence;
    }

    /**
     * 阻塞到下一个毫秒，直到获得新的时间戳
     *
     * @param lastTimestamp 上次生成 ID 的时间截
     * @return 当前时间戳(毫秒)
     */
    protected long tilNextMillis(long lastTimestamp) {
        long timestamp = timeGen();

        while (timestamp <= lastTimestamp) {
            timestamp = timeGen();
        }

        return timestamp;
    }

    /**
     * 返回当前时间，以毫秒为单位
     *
     * @return 当前时间(毫秒)
     */
    protected long timeGen() {
        return System.currentTimeMillis();
    }

    public static void main(String[] args) {
        IdWorker idWorker = new IdWorker(0, 0);

        for (int i = 0; i < 1000; i++) {
            long id = idWorker.nextId();
            System.out.println(id);
            System.out.println(StringUtils.leftPad(Long.toBinaryString(id), 64, "0"));
        }
    }
}
```

可能你要问，哎呀，只能保证 70 内不重复，70 年后怎么办呢？我的答案是，让 70 年后公司的同事去头疼吧，问题是，你们公司能活到什么时候都是个问题！

> 提示: 求一个整数，它的二进制表示中最后 n 位为 1: `-1 ^ (-1 << n)` 或者 `~(~0<<n)`
> 分析: -1 的二进制表示中所有位都是 1，`-1 << n` 的结果为前 32-n 位为 1，后 n 位为 0，与 -1 执行取反操作得到的结果，前 32-n 位为 0，后 n 为 1(取反: 相同为 0，不同为 1)。
>
> ```java
> import org.apache.commons.lang.StringUtils;
>
> public class Test {
>     public static void main(String[] args) throws Exception {
>         printBinary(lastNBits1(3));
>         printBinary(lastNBits1(8));
>         printBinary(lastNBits1(20));
>         printBinary(lastNBits1(31));
>     }
>
>     public static int lastNBits1(int n) {
>         return -1 ^ (-1 << n);
>     }
>
>     public static void printBinary(int n) {
>         String bs = Integer.toBinaryString(n);
>         System.out.println(StringUtils.leftPad(bs, 32, '0'));
>     }
> }
> ```
>
> 输出:
>
> ```
> 00000000000000000000000000000111
> 00000000000000000000000011111111
> 00000000000011111111111111111111
> 01111111111111111111111111111111
> ```

