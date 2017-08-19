---
title: 测试 ThreadLocal
date: 2017-08-16 23:46:01
tags: Java
---
每个 Thread 都有一个 ThreadLocalMap 的对象，存储时以 ThreadLocal 变量为 key，`set()` 的参数作为 value，这样同一个 ThreadLocal 变量在不同的线程中就可以存储不同的数据。

```java
public class ThreadLocalTest {
    private static ThreadLocal<String> foo = new ThreadLocal<>();

    public static void main(String[] args) throws Exception {
        new Thread(() -> {
            await(300);
            System.out.println(Thread.currentThread().getName() + ": " + foo.get()); // [2] 输出: Thread-1: null
            foo.set("1"); // [3]

            await(1000);
            System.out.println(Thread.currentThread().getName() + ": " + foo.get()); // [5] 输出: Thread-1: 1
        }, "Thread-1").start();

        new Thread(() -> {
            foo.set("2"); // [1]

            await(600);
            System.out.println(Thread.currentThread().getName() + ": " + foo.get()); //[4] 输出: Thread-2: 2
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

>ThreadLocal 的变量一般定义为 private static 的。