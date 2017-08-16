---
title: 测试 ThreadLocal
date: 2017-08-16 23:46:01
tags: Java
---
ThreadLocal 变量以线程信息作为 key，`set()` 的值作为 value，这样同一个变量在不同的线程中就可以存储不同的数据。

```java
public class ThreadLocalTest {
    public ThreadLocal<String> foo = new ThreadLocal<>();

    public static void main(String[] args) throws Exception {
        ThreadLocalTest test = new ThreadLocalTest();

        new Thread(() -> {
            await(300);
            System.out.println(Thread.currentThread().getName() + ": " + test.foo.get()); // [2] 输出: Thread-1: null
            test.foo.set("1"); // [3]

            await(1000);
            System.out.println(Thread.currentThread().getName() + ": " + test.foo.get()); // [5] 输出: Thread-1: 1
        }, "Thread-1").start();

        new Thread(() -> {
            test.foo.set("2"); // [1]

            await(600);
            System.out.println(Thread.currentThread().getName() + ": " + test.foo.get()); //[4] 输出: Thread-2: 2
        }, "Thread-2").start();
    }

    public static void await(long timeout) {
        try { Thread.sleep(timeout); } catch (InterruptedException e) {}
    }
}
```
输出:

```
Thread-1: null
Thread-2: 2
Thread-1: 1
```

