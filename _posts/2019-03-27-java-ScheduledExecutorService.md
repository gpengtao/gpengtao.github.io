---
layout: post
title: ScheduledExecutorService使用上的小坑
categories: java
description: ScheduledExecutorService使用上的小坑。
keywords: ScheduledExecutorService
---

*目录*
* Toc
{:toc}

```java
@Test
public void test_schedule() throws IOException {
    ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);

    AtomicInteger integer = new AtomicInteger(0);
    executor.scheduleAtFixedRate(() -> {

        System.out.println(integer.incrementAndGet());

        if (integer.intValue() == 3) {
            throw new RuntimeException();
        }
    }, 0, 1, TimeUnit.SECONDS);

    System.in.read();
}
```
执行结果：
> 1<br>
> 2<br>
> 3<br>

这个代码有两个问题：

1、出现的 RuntimeException 不会有任何感知。

因为没有 try catch 然后打印栈信息，也没有记录 log 打印栈信息，所以这个地方不会有任何感知。不要想着 java 会给你打印栈信息，java 的线程没干这个事情。

2、RuntimeException 会导致定时任务失败，执行3次之后终止了，再也不执行了。

看下 scheduleAtFixedRate 的注释：
<p>
If any execution of the task encounters an exception, subsequent executions are suppressed. Otherwise, the task will only terminate via cancellation or termination of the executor.
</p>
意思是：如果该任务的执行遇到异常，则随后的执行将被禁止。否则，任务将只能通过取消或终止执行者而终止。

正确的姿势：
```java
new Runnable() {
    @Override
    public void run() {
        try {

        } catch (Throwable t) {
            // do log or something
        }
    }
};
```
**注意：**
要 try catch Throwable，如果 try catch 了 RuntimeException 可能产生的非运行时异常导致你线程死掉，而且**没有任何感知**。