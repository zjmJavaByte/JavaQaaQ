## Java线程池中的各个参数如何合理设置

## 一、前言

​		在开发过程中，好多场景要用到线程池。每次都是自己根据业务场景来设置线程池中的各个参数。

​		这两天又有需求碰到了，索性总结一下方便以后再遇到可以直接看着用。

​		虽说根据业务场景来设置各个参数的值，但有些万变不离其宗，掌握它的原理对如何用好线程池起了至关重要的作用。

那我们接下来就来进行线程池的分析。

## 二、ThreadPoolExecutor的重要参数

我们先来看下`ThreadPoolExecutor的带的那些重要参数的构造器。

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    ...
}
```

### 1、corePoolSize: 核心线程数

这个应该是最重要的参数了，所以如何合理的设置它十分重要。

- 核心线程会一直存活，及时没有任务需要执行。
- 当线程数小于核心线程数时，即使有线程空闲，线程池也会优先创建新线程处理。
- 设置`allowCoreThreadTimeout=true`（默认false）时，核心线程会超时关闭。

如何设置好的前提我们要很清楚的知道`CPU`密集型和`IO`密集型的区别。

**(1)、CPU密集型**

​		`CPU`密集型也叫计算密集型，指的是系统的硬盘、内存性能相对`CPU`要好很多，此时，系统运作大部分的状况是`CPU Loading 100%，CPU`要读/写`I/O`(硬盘/内存)，`I/O`在很短的时间就可以完成，而`CPU`还有许多运算要处理，`CPU Loading `很高。

​		在多重程序系统中，大部分时间用来做计算、逻辑判断等`CPU`动作的程序称之`CPU bound`。例如一个计算圆周率至小数点一千位以下的程序，在执行的过程当中绝大部分时间用在三角函数和开根号的计算，便是属于`CPU bound`的程序。

`CPU bound`的程序一般而言`CPU`占用率相当高。这可能是因为任务本身不太需要访问`I/O`设备，也可能是因为程序是多线程实现因此屏蔽掉了等待`I/O`的时间。

**(2)、IO密集型**

​		`IO`密集型指的是系统的`CPU`性能相对硬盘、内存要好很多，此时，系统运作，大部分的状况是`CPU`在等`I/O` (硬盘/内存) 的读/写操作，此时`CPU Loading`并不高。

​		`I/O bound`的程序一般在达到性能极限时，`CPU`占用率仍然较低。这可能是因为任务本身需要大量`I/O`操作，而`pipeline`做得不是很好，没有充分利用处理器能力。

好了，了解完了以后我们就开搞了。

**(3)、先看下机器的CPU核数，然后在设定具体参数：**

自己测一下自己机器的核数

```java
System.out.println(Runtime.getRuntime().availableProcessors());
```

即CPU核数 `= Runtime.getRuntime().availableProcessors()`

**(4)、分析下线程池处理的程序是CPU密集型还是IO密集型**

CPU密集型：`corePoolSize = CPU核数 + 1`

> 这里，之所以线程池的大小比 CPU 的个数还多1个，是因为考虑到即便是 CPU 密集型的任务其执行线程也可能在某一时刻由于某种原因，如缺页中桥（Page Fault）而出现等待。此时，一个额外的线程可以继续使用 CPU 时间片。

IO密集型：`corePoolSize = CPU核数 * 2`

> 另外，对于`I/O` 密集型任务我们需要注意 `I/O`  操作会引起上下文切换
> 这就意味着进行 `I/O` 操作的线程越多由`I/O`  操作引起的上下文切换也越多。因此，对于`I/O` 密集型任务不妨将相应的线程池的核心线程池大小（`Core Pool Size`） 设置为1，并将其最大线程池大小（`Maximum Pool Size`）设置为 2x N

**(5)混合型**

​		如果可以拆分，将其拆分成一个`CPU`密集型任务 和一个`IO`密集型任务，只要这两个任务执行的时间相差不是太大，那么分解后执行的吞吐量 将高于串行执行的吞吐量。如果这两个任务执行时间相差太大，则没必要进行分解。

### 2、maximumPoolSize：最大线程数

- `当线程数>=corePoolSize`，且任务队列已满时。线程池会创建新线程来处理任务。
- `当线程数=maxPoolSize`，且任务队列已满时，线程池会拒绝处理任务而抛出异常。

### 3、keepAliveTime：线程空闲时间

- 当线程空闲时间达到`keepAliveTime`时，线程会退出，直到线程数量`=corePoolSize`。
- 如果`allowCoreThreadTimeout=true`，则会直到线程数量=0。

### 4、queueCapacity：任务队列容量（阻塞队列）

- 当核心线程数达到最大时，新任务会放在队列中排队等待执行

### 5、allowCoreThreadTimeout：允许核心线程超时

### 6、rejectedExecutionHandler：任务拒绝处理器

两种情况会拒绝处理任务：

- 当线程数已经达到`maxPoolSize`，且队列已满，会拒绝新任务。
- 当线程池被调用`shutdown()`后，会等待线程池里的任务执行完毕再`shutdown`。如果在调用`shutdown()`和线程池真正`shutdown`之间提交任务，会拒绝新任务。

线程池会调用`rejectedExecutionHandler`来处理这个任务。如果没有设置默认是`AbortPolicy`，会抛出异常。

`ThreadPoolExecuto`r 采用了策略的设计模式来处理拒绝任务的几种场景。

这几种策略模式都实现了`RejectedExecutionHandler `接口。

- `AbortPolicy` 丢弃任务，抛运行时异常。
- `CallerRunsPolicy `执行任务。
- `DiscardPolicy `忽视，什么都不会发生。
- `DiscardOldestPolicy` 从队列中踢出最先进入队列（最后一个执行）的任务。

## 三、如何设置参数

默认值：

```
corePoolSize = 1
maxPoolSize = Integer.MAX_VALUE
queueCapacity = Integer.MAX_VALUE
keepAliveTime = 60s
allowCoreThreadTimeout = false
rejectedExecutionHandler = AbortPolicy()
```

**如何来设置呢？**

需要根据几个值来决定

`tasks` ：每秒的任务数，假设为500~1000

`taskcost`：每个任务花费时间，假设为0.1s

`esponsetime`：系统允许容忍的最大响应时间，假设为1s

**做几个计算**

`corePoolSize` = 每秒需要多少个线程处理？

`threadcount = tasks/(1/taskcost) = tasks*taskcout = (500 ~ 1000)*0.1 = 50~100 `个线程。

corePoolSize设置应该大于50。

根据8020原则，如果80%的每秒任务数小于800，那么corePoolSize设置为80即可。

```
queueCapacity = (coreSizePool/taskcost) * responsetime
```

计算可得 `queueCapacity = 80/0.1*1 = 800`。意思是队列里的线程可以等待1s，超过了的需要新开线程来执行。

切记不能设置为`Integer.MAX_VALUE`，这样队列会很大，线程数只会保持在`corePoolSize`大小，当任务陡增时，不能新开线程来执行，响应时间会随之陡增。

`maxPoolSize` 最大线程数在生产环境上我们往往设置成`corePoolSiz`一样，这样可以减少在处理过程中创建线程的开销。

`rejectedExecutionHandler`：根据具体情况来决定，任务不重要可丢弃，任务重要则要利用一些缓冲机制来处理。

`keepAliveTime和allowCoreThreadTimeout`采用默认通常能满足。

以上都是理想值，实际情况下要根据机器性能来决定。如果在未达到最大线程数的情况机器`cpu load`已经满了，则需要通过升级硬件和优化代码，降低`taskcost`来处理。

**以下是我自己的的线程池配置：**

```java
@Configuration
public class ConcurrentThreadGlobalConfig {
    @Bean
    public ThreadPoolTaskExecutor defaultThreadPool() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        //核心线程数目
        executor.setCorePoolSize(65);
        //指定最大线程数
        executor.setMaxPoolSize(65);
        //队列中最大的数目
        executor.setQueueCapacity(650);
        //线程名称前缀
        executor.setThreadNamePrefix("DefaultThreadPool_");
        //rejection-policy：当pool已经达到max size的时候，如何处理新任务
        //CALLER_RUNS：不在新线程中执行任务，而是由调用者所在的线程来执行
        //对拒绝task的处理策略
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        //线程空闲后的最大存活时间
        executor.setKeepAliveSeconds(60);
        //加载
        executor.initialize();
        return executor;
    }
}

```

## 四、线程池队列的选择

workQueue - 当线程数目超过核心线程数时用于保存任务的队列。主要有3种类型的BlockingQueue可供选择：无界队列，有界队列和同步移交。从参数中可以看到，此队列仅保存实现Runnable接口的任务。

这里再重复一下新任务进入时线程池的执行策略：

- 当正在运行的线程小于corePoolSize，线程池会创建新的线程。
- 当大于corePoolSize而任务队列未满时，就会将整个任务塞入队列。
- 当大于corePoolSize而且任务队列满时，并且小于maximumPoolSize时，就会创建新额线程执行任务。
- 当大于maximumPoolSize时，会根据handler策略处理线程。

### 1、无界队列

队列大小无限制，常用的为无界的LinkedBlockingQueue，使用该队列作为阻塞队列时要尤其当心，当任务耗时较长时可能会导致大量新任务在队列中堆积最终导致OOM。

阅读代码发现，Executors.newFixedThreadPool 采用就是 LinkedBlockingQueue，而博主踩到的就是这个坑，当QPS很高，发送数据很大，大量的任务被添加到这个无界LinkedBlockingQueue 中，导致cpu和内存飙升服务器挂掉。

当然这种队列，maximumPoolSize 的值也就无效了。

当每个任务完全独立于其他任务，即任务执行互不影响时，适合于使用无界队列；例如，在 Web 页服务器中。

这种排队可用于处理瞬态突发请求，当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。

### 2、有界队列

当使用有限的 maximumPoolSizes 时，有界队列有助于防止资源耗尽，但是可能较难调整和控制。

常用的有两类，一类是遵循FIFO原则的队列如ArrayBlockingQueue，另一类是优先级队列如PriorityBlockingQueue。

PriorityBlockingQueue中的优先级由任务的Comparator决定。

使用有界队列时队列大小需和线程池大小互相配合，线程池较小有界队列较大时可减少内存消耗，降低cpu使用率和上下文切换，但是可能会限制系统吞吐量。

### 3、同步移交队列

如果不希望任务在队列中等待而是希望将任务直接移交给工作线程，可使用SynchronousQueue作为等待队列。

SynchronousQueue不是一个真正的队列，而是一种线程之间移交的机制。要将一个元素放入SynchronousQueue中，必须有另一个线程正在等待接收这个元素。

只有在使用无界线程池或者有饱和策略时才建议使用该队列。

[引用文章](https://www.jb51.net/article/215312.htm)