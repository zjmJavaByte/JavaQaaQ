**线程池与Executor框架**

#### 前言

​		诸如 Web 服务器、数据库服务器、文件服务器或邮件服务器之类的许多服务器应用程序都面向处理来自某些远程来源的大量短小的任务。请求以某种方式到达服务器，这种方式可能是通过网络协议（例如 HTTP、FTP ）、通过 JMS队列或者可能通过轮询数据库。不管请求如何到达，服务器应用程序中经常出现的情况是：单个任务处理的时间很短而请求的数目却是巨大的。每当一个请求到达就创建一个新线程，然后在新线程中为请求服务，但是频繁的创建线程，销毁线程所带来的系统开销其实是非常大的。

​		线程池为线程生命周期开销问题和资源不足问题提供了解决方案。通过对多个任务重用线程，线程创建的开销被分摊到了多个任务上。其好处是，因为在请求到达时线程已经存在，所以无意中也消除了线程创建所带来的延迟。这样，就可以立即为请求服务，使应用程序响应更快。而且，通过适当地调整线程池中的线程数目，也就是当请求的数目超过某个阈值时，就强制其它任何新到的请求一直等待，直到获得一个线程来处理为止，从而可以防止资源不足。

​		用线程池构建的应用程序容易遭受任何其它多线程应用程序容易遭受的所有并发风险，诸如同步错误和死锁，它还容易遭受特定于线程池的少数其它风险，诸如与池有关的死锁、资源不足和线程泄漏。

#### 创建线程池及其使用

- 查看其构造函数

```java
public ThreadPoolExecutor(int corePoolSize,//核心线程池
                              int maximumPoolSize,//线程池最大容量
                              long keepAliveTime,//当线程数量大于核心时，多余的空闲线程在终止之前等待新任务的最大时间
                              TimeUnit unit,//时间单位
                              BlockingQueue<Runnable> workQueue,//工作队列
                              ThreadFactory threadFactory,//线程工厂
                              RejectedExecutionHandler handler//当队伍中的任务满载后使用的拒绝策略
                         ) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

- 构建线程池

```java
package com.zjm.Multithreading.part9;
import cn.hutool.core.thread.ThreadFactoryBuilder;
import java.util.concurrent.*;
/**
 * @author zjm
 * @version 1.0
 * 描述：线程池的创建
 * @date 2020/9/26 12:25
 */
public class ThreadPoolDemo {
    //给每一个线程创建名称
    private static ThreadFactory namedThreadFactory = new ThreadFactoryBuilder()
            .setNamePrefix("demo-pool-%d").build();
    private static ExecutorService singleThreadPool = new ThreadPoolExecutor(1, 1,
            0L, TimeUnit.MILLISECONDS,
            //基于链表的有界/无界阻塞队列(没有设置就是无界)，队列容量为10                  
            new LinkedBlockingQueue<Runnable>(1024), namedThreadFactory,
            //饱和策略抛出异常，拒绝任务提交 
            new ThreadPoolExecutor.AbortPolicy());
    public static void main(String[] args) {
        //有返回值
    Future<Integer> future = singleThreadPool.submit(() -> {
        Thread.sleep(1000L * 10);
        return 2 * 5;
    });
    //直到有返回值才会继续往下执行
    Integer num = future .get();
    System.out.println(s);
        //没有返回值
        singleThreadPool.execute(()-> System.out.println(Thread.currentThread().getName()));
        singleThreadPool.shutdown();
    }
}
```

#### Callable与FutureTask



#### 线程池的核心组成部分及其运行机制

#### 线程池拒绝策略

#### Executors框架

#### 队列

