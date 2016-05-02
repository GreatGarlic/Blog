---
title: Future
date: 2016-05-02 22:27:55
tags: Java
---

Future 的作用

* 作为 `ExecutorService.submit(Callable|Runnable)` 的返回结果
* 得到了 Future，说明其相关的任务已经提交给线程池去执行了
* 获取任务的结果(阻塞): `Future.get()`
* 取消任务: `Future.cancel()`
* 查看任务是否完成: `Future.isDone()`

> 如果线程池执行的任务没有返回结果，直接用 Runnable 就好了，不需要用 Callable (代码里加上无意义的返回语句有点奇怪)，Callable 更多是任务执行后有结果返回。

<!--more-->

```java
import java.util.concurrent.*;

public class FutureTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executor = Executors.newFixedThreadPool(1);

        // Future 需要和线程池一起使用
        // 得到 Future, 说明其相关的任务已经提交给线程池去执行了
        // Future 用来查看是否已经完成, 取消任务, 获取任务的结果
        Future<Integer> future = executor.submit(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                int sum = 0;
                for (int i = 0; i < 100; ++i) {
                    sum += i;
                    Thread.sleep(10);
                }

                return sum;
            }
        });

        executor.shutdown(); // 不再接受新的任务, 已提交的任务会继续执行完成

        System.out.println("===> main");
        System.out.println(future.get()); // Block
    }
}
```

## 参考
* [Java并发编程: Callable、Future 和 FutureTask](http://www.cnblogs.com/dolphin0520/p/3949310.html)
