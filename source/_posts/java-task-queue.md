---
title: 任务队列
date: 2017-05-28 15:28:01
tags: [Java, Util]
---

可以使用 Java 提供的线程池简单地实现一个任务队列:

```java
package com.xtuer.util;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * 任务队列，同时执行任务的数量由构造函数的参数 concurrentTaskCount 指定。
 */
public class TaskQueue {
    private ExecutorService executor;

    /**
     * 创建任务队列，concurrentTaskCount 指定同时执行任务的数量。
     * 有些情况下任务需要排队一个执行完后再执行另一个，此时 concurrentTaskCount 传入 1。
     *
     * @param concurrentTaskCount 同时执行任务的数量
     */
    public TaskQueue(int concurrentTaskCount) {
        executor = Executors.newFixedThreadPool(concurrentTaskCount);
    }

    /**
     * 添加任务，根据不同的业务逻辑定义一个任务类，继承自 Runnable，
     * 可以在属性中存储任务相关的数据，在 run() 中实现任务逻辑。
     * 当然也可以重载 addTask() 函数实现添加不同的任务。
     *
     * @param task
     */
    public void addTask(Runnable task) {
        executor.submit(task);
    }

    /**
     * 下面的实现是为了测试使用
     *
     * @param n 任务内容
     * @param delay 任务消耗的时间，单位为秒，为了测试用的
     */
    public void addTask(int n, int delay) {
        addTask(() -> {
            // 模拟任务执行消耗时间
            try {
                Thread.sleep(delay * 1000);
            } catch (InterruptedException e) {
            }

            System.out.println(n + " started at " + System.currentTimeMillis() + " and elapsed " + delay * 1000);
        });
    }

    /**
     * 销毁任务队列，不再接受新的任务。
     * Spring bean 的 destroy-method 函数。
     */
    public void destroy() {
        executor.shutdown();
    }

    public static void main(String[] args) throws Exception {
        TaskQueue taskQueue = new TaskQueue(1);

        taskQueue.addTask(1, 1);
        taskQueue.addTask(2, 1);
        taskQueue.addTask(3, 1);
        taskQueue.addTask(4, 1);
        taskQueue.addTask(5, 1);

        taskQueue.destroy();
    }
}
```

可以如下使用 Spring bean 来生成任务队列的对象

```xml
<!--单任务队列-->
<bean id="singleTaskQueue" class="com.xtuer.util.TaskQueue" destroy-method="destroy">
    <constructor-arg value="1"/>
</bean>

<!--多任务队列-->
<bean id="multiTaskQueue" class="com.xtuer.util.TaskQueue" destroy-method="destroy">
    <constructor-arg value="222"/>
</bean>
```

然后在 Controller 中如下使用

```java
@Resource(name="singleTaskQueue")
private TaskQueue singleTaskQueue;

@GetMapping("/tasks/{taskId}")
@ResponseBody
public Result task(@PathVariable int taskId) {
    Random rand = new Random();
    singleTaskQueue.addTask(taskId, rand.nextInt(4) + 1); // 任务执行时间为 1 到 4 秒

    return Result.ok("" + taskId);
}
```

完全自己实现的话，任务队列继承 Thread，用一个 list 存储任务，在 run() 函数中用循环查看是否有任务可执行，如果没有则调用 wait() 等待，当调用 addTask() 添加新的任务后调用 notify() 让 while 循环中可获取一个任务执行，获取和添加任务时还要锁住队列等，如果同时允许执行多个任务则还要用一个计数器记录正在执行的任务数，需要处理好各种细节。使用 Executors.newFixedThreadPool() 后，这些细节都不需要我们关心了。