---
title: 多线程&线程池
date: 2020-02-27 22:32:50
tags:
    - java
categories:
    - java
---

### 多线程&线程池

#### 多线程

##### 相关API

* start()：启动当前线程，调用线程中的run方法
* run()：通常需要重写Thread类中的此方法，将创建的线程要执行的操作声明在此方法中
* currentThread()：静态方法，返回执行当前代码的线程
* getName()：获取当前线程的名字（Thread.currentThread().getName()）
* setName()：设置当前线程的名字
* yield()：主动释放当前线程的执行权
* join()：在线程中插入执行另一个线程，该线程被阻塞，直到插入执行的线程完全执行完毕以后，该线程才继续执行下去
* stop()：过时方法。当执行此方法时，强制结束当前线程。
* sleep（long millis）：线程休眠一段时间（单位ms）
* isAlive（）：判断当前线程是否存活

##### 线程的调度

* 调度策略
    * 时间片：线程的调度采用时间片轮转的方式
    * 抢占式：高优先级的线程抢占CPU
* Java的调度方法
    * 对于同优先级的线程组成先进先出队列（先到先服务），使用时间片策略
    * 对高优先级，使用优先调度的抢占式策略

##### 线程的状态

* 创建：当new了一个线程，并没有调用start之前，线程处于创建状态；
* 就绪：当调用了start之后，线程处于就绪状态，这是，线程调度程序还没有设置执行当前线程；
* 运行：线程调度程序执行到线程时，当前线程从就绪状态转成运行状态，开始执行run方法里边的代码；
* 阻塞：线程在运行的时候，被暂停执行（通常等待某项资源就绪后在执行，sleep、wait可以导致线程阻塞），这是该线程处于阻塞状态；
* 死亡：当一个线程执行完run方法里边的代码或调用了stop方法后，该线程结束运行

##### 线程的优先级

* 等级
    * MAX_PRIORITY：10
    * MIN_PRIORITY：1
    * NORM_PRIORITY：5
* 方法：
    * getPriority()：返回线程优先级
    * setPriority(int newPriority)：改变线程的优先级

##### 创建方式

* 继承于Thread类
```
public class ThreadTest extends Thread {
    @Override
    public void run() {
        System.out.println("run");
    }

    public static void main(String[] args) {
        new ThreadTest().start();
    }
}
```

* 实现Runnable接口
```
public class ThreadTest implements Runnable {
    @Override
    public void run() {
        System.out.println("run");
    }

    public static void main(String[] args) {
        new Thread(new ThreadTest()).start();
        // 或者
        new Thread(() -> System.out.println("run")).start();
    }
}
```

* 实现Callable接口（带返回值）
```
import java.util.concurrent.Callable;
import java.util.concurrent.FutureTask;

public class ThreadTest implements Callable<String> {

    @Override
    public String call() throws Exception {
        System.out.println("call");
        return "success";
    }

    public static void main(String[] args) throws Exception {
        ThreadTest callable = new ThreadTest();
        FutureTask<String> futureTask = new FutureTask<>(callable);
        Thread thread = new Thread(futureTask);
        thread.start();
        System.out.println(futureTask.get());
    }
}
```

#### 线程池

* 背景：经常创建和销毁，使用量特别大的资源，比如并发情况下的线程，对性能影响很大。
* 思路：提前创建好多个线程，放入线程池之，使用时直接获取，使用完放回池中。可以避免频繁创建销毁，实现重复利用。类似生活中的公共交通工具。（数据库连接池）
* 好处：提高响应速度；减少了创建新线程的时间；降低资源消耗（重复利用线程池中线程，不需要每次都创建）；便于线程管理

##### 相关API
ExecutorService 和 Executors
* ExecutorService：真正的线程池接口。常见子类ThreadPoolExecutor.
* void execute(Runnable command)：执行任务/命令，没有返回值，一般用来执行Runnable
* Future submit(Callable task)：执行任务，有返回值，一般用来执行Callable
* void shutdown()：关闭连接池。

##### 配置参数

* corePoolSize：线程池的大小。线程池创建之后不会立即去创建线程，而是等待线程的到来。当当前执行的线程数大于改值是，线程会加入到缓冲队列。
* maximumPoolSize：线程池中创建的最大线程数。
* keepAliveTime：空闲的线程多久时间后被销毁。默认情况下，改值在线程数大于corePoolSize时，对超出corePoolSize值得这些线程起作用。
* unit：TimeUnit枚举类型的值，代表keepAliveTime时间单位，可以取下列值：
    * TimeUnit.HOURS; //小时
    * TimeUnit.MINUTES; //分钟
    * TimeUnit.SECONDS; //秒
    * TimeUnit.MILLISECONDS; //毫秒
    * TimeUnit.MICROSECONDS; //微妙
    * TimeUnit.NANOSECONDS; //纳秒
* workQueue：阻塞队列，用来存储等待执行的任务，决定了线程池的排队策略，有以下取值：
    * ArrayBlockingQueue;
    * LinkedBlockingQueue;
    * SynchronousQueue;
    * threadFactory：线程工厂，是用来创建线程的。默认new Executors.DefaultThreadFactory();
* handler：线程拒绝策略。当创建的线程超出maximumPoolSize，且缓冲队列已满时，新任务会拒绝，有以下取值：
    * ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。 
    * ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。 
    * ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）。
    * ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务。

##### 创建方式
|  Executors   | 工具类，线程池的工厂类，用于创建并返回不同类型的线程池  |
|  ----  | ----  |
|  Executors.newCachedThreadPool()  |	创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。  |
|  Executors.newFixedThreadPool(n)  |	创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。  |
|  Executors.newSingleThreadExecutor()  |	创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。  |
|  Executors.newScheduledThreadPool(n)  |	创建一个定长线程池，支持定时及周期性任务执行。  |

```
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class ThreadTest implements Callable<String> {
    @Override
    public String call() throws Exception {
        System.out.println("call");
        return "success";
    }

    static class ThreadTest2 implements Runnable {
        @Override
        public void run() {
            System.out.println("run");
        }
    }

    public static void main(String[] args) throws Exception {
        // 创建固定线程个数为十个的线程池
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        ThreadTest callable = new ThreadTest();
        ThreadTest2 runnable = new ThreadTest2();
        // 执行线程
        // 适合适用于Runnable
        executorService.execute(runnable);
        // 适合使用于Callable
        Future<String> submit = executorService.submit(callable);
        System.out.println(submit.get());
        // 关闭线程池
        executorService.shutdown();
    }
}
```
