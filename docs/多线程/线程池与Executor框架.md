# 前言

​		诸如 Web 服务器、数据库服务器、文件服务器或邮件服务器之类的许多服务器应用程序都面向处理来自某些远程来源的大量短小的任务。请求以某种方式到达服务器，这种方式可能是通过网络协议（例如 HTTP、FTP ）、通过 JMS队列或者可能通过轮询数据库。不管请求如何到达，服务器应用程序中经常出现的情况是：单个任务处理的时间很短而请求的数目却是巨大的。每当一个请求到达就创建一个新线程，然后在新线程中为请求服务，但是频繁的创建线程，销毁线程所带来的系统开销其实是非常大的。

​		线程池为线程生命周期开销问题和资源不足问题提供了解决方案。通过对多个任务重用线程，线程创建的开销被分摊到了多个任务上。其好处是，因为在请求到达时线程已经存在，所以无意中也消除了线程创建所带来的延迟。这样，就可以立即为请求服务，使应用程序响应更快。而且，通过适当地调整线程池中的线程数目，也就是当请求的数目超过某个阈值时，就强制其它任何新到的请求一直等待，直到获得一个线程来处理为止，从而可以防止资源不足。

​		用线程池构建的应用程序容易遭受任何其它多线程应用程序容易遭受的所有并发风险，诸如同步错误和死锁，它还容易遭受特定于线程池的少数其它风险，诸如与池有关的死锁、资源不足和线程泄漏。

​		在开发过程中，合理地使用线程池能够带来3个好处。

- **降低资源消耗**。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。 
- **提高响应速度**。当任务到达时，任务可以不需要等到线程创建就能立即执行。

- **提高线程的可管理性**。线程是稀缺资源，如果无限制地创建，不仅会消耗系统资源， 还会降低系统的稳定性，使用线程池可以进行统一分配、调优和监控。但是，要做到合理利用 线程池，必须对其实现原理了如指掌。

# 线程池的实现原理

线程池的主要处理流程，处理流程图如图所示

![image-20220516094354747](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205160944483.png)

从图中可以看出，当提交一个新任务到线程池时，线程池的处理流程如下：

- 线程池判断核心线程池里的线程是否都在执行任务。如果不是，则创建一个新的工作线程来执行任务。如果核心线程池里的线程都在执行任务，则进入下个流程。
- 线程池判断工作队列是否已经满。如果工作队列没有满，则将新提交的任务存储在这 个工作队列里。如果工作队列满了，则进入下个流程。
- 线程池判断线程池的线程是否都处于工作状态。如果没有，则创建一个新的工作线程 来执行任务。如果已经满了，则交给饱和策略来处理这个任务。

我们在拿到具体的代码中查看：

```java
private static ExecutorService singleThreadPool = new ThreadPoolExecutor(3, 6,
            0L, TimeUnit.MILLISECONDS,
            //基于链表的有界/无界阻塞队列(没有设置就是无界)，队列容量为10                  
            new LinkedBlockingQueue<Runnable>(1024), namedThreadFactory,
            //饱和策略抛出异常，拒绝任务提交 
            new ThreadPoolExecutor.AbortPolicy());
public static void main(String[] args) {
        singleThreadPool.execute(()-> System.out.println(Thread.currentThread().getName()));
        singleThreadPool.shutdown();
    }
```

如上代码`ThreadPoolExecutor`的执行示意图如下：

![image-20220516094937426](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205160949457.png)

`ThreadPoolExecutor`执行`execute`方法分下面4种情况：

- 如果当前运行的线程少于`corePoolSize`，则创建新线程来执行任务(注意，执行这一步骤 需要获取全局锁)。

- 如果运行的线程等于或多于`corePoolSize`，则将任务加入`BlockingQueue`。 

- 如果无法将任务加入`BlockingQueue`(队列已满)，则创建新的线程来处理任务(注意，执行这一步骤需要获取全局锁)。 4)如果创建新线程将使当前运行的线程超出`maximumPoolSize`，任务将被拒绝，并调用

`RejectedExecutionHandler.rejectedExecution()`方法。

​		`ThreadPoolExecutor`采取上述步骤的总体设计思路，是为了在执行`execute()`方法时，尽可能 地避免获取全局锁(那将会是一个严重的可伸缩瓶颈)。在`ThreadPoolExecutor`完成预热之后 (当前运行的线程数大于等于`corePoolSize`)，几乎所有的`execute()`方法调用都是执行步骤2，而步骤2不需要获取全局锁。

# 创建线程池及其使用

## 查看其构造函数

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

## 线程池的核心组成部分

- `corePoolSize`(线程池的基本大小):当提交一个任务到线程池时，线程池会创建一个线 程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，等到需要执行的任 务数大于线程池基本大小时就不再创建。如果调用了线程池的`prestartAllCoreThreads()`方法， 线程池会提前创建并启动所有基本线程。
- `runnableTaskQueue`(任务队列):用于保存等待执行的任务的阻塞队列。可以选择以下几 个阻塞队列。
  - `ArrayBlockingQueue`:是一个基于数组结构的有界阻塞队列，此队列按FIFO(先进先出)原 则对元素进行排序。
  - `LinkedBlockingQueue`:一个基于链表结构的阻塞队列，此队列按FIFO排序元素，吞吐量通 常要高于`ArrayBlockingQueue`。静态工厂方法`Executors.newFixedThreadPool()`使用了这个队列
  - `SynchronousQueue`:一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用 移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于`LinkedBlockingQueue`，静态工 厂方法`Executors.newCachedThreadPool`使用了这个队列。
  - `PriorityBlockingQueue`:一个具有优先级的无限阻塞队列。
- `maximumPoolSize`(线程池最大数量):线程池允许创建的最大线程数。如果队列满了，并 且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。值得注意的是，如 果使用了无界的任务队列这个参数就没什么效果。
- `ThreadFactory`:用于设置创建线程的工厂，可以通过线程工厂给每个创建出来的线程设 置更有意义的名字。使用开源框架`guava`提供的`ThreadFactoryBuilder`可以快速给线程池里的线 程设置有意义的名字

```java
new ThreadFactoryBuilder().setNameFormat("XX-task-%d").build();
```

- `RejectedExecutionHandler`(饱和策略):当队列和线程池都满了，说明线程池处于饱和状 态，那么必须采取一种策略处理提交的新任务。这个策略默认情况下是`AbortPolicy`，表示无法 处理新任务时抛出异常。在JDK 1.5中Java线程池框架提供了以下4种策略。
  - `AbortPolicy`:直接抛出异常。
  - `CallerRunsPolicy`:只用调用者所在线程来运行任务。
  - `DiscardOldestPolicy`:丢弃队列里最近的一个任务，并执行当前任务。
  - `DiscardPolicy`:不处理，丢弃掉。

> 也可以根据应用场景需要来实现`RejectedExecutionHandler`接口自定义策略。如记录 日志或持久化存储不能处理的任务。

- `keepAliveTime`(线程活动保持时间):线程池的工作线程空闲后，保持存活的时间。所以， 如果任务很多，并且每个任务执行的时间比较短，可以调大时间，提高线程的利用率。
- `TimeUnit`(线程活动保持时间的单位):可选的单位有天(`DAYS`)、小时`(HOURS`)、分钟 (`MINUTES`)、毫秒(`MILLISECONDS`)、微秒(`MICROSECONDS`，千分之一毫秒)和纳秒(`NANOSECONDS`，千分之一微秒)。

## 构建线程池

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



#### 

#### 线程池拒绝策略

#### Executors框架

#### 队列

