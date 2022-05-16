# 什么是Fork/Join框架

​		`Fork/Join`框架是`Java 7`提供的一个用于并行执行任务的框架，是一个把大任务分割成若干个小任务，最终汇总每个小任务结果后得到大任务结果的框架。

# 工作窃取算法

- 工作窃取`(work-stealing`)算法是指某个线程从其他队列里窃取任务来执行。

- 为了减少窃取任务线程和被 窃取任务线程之间的竞争，通常会使用双端队列，被窃取任务线程永远从双端队列的头部拿 任务执行，而窃取任务的线程永远从双端队列的尾部拿任务执行。

**流程图如下**

![image-20220513110238335](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205131102367.png)

**工作窃取算法的优点**:充分利用线程进行并行计算，减少了线程间的竞争。

**工作窃取算法的缺点**:在某些情况下还是存在竞争，比如双端队列里只有一个任务时。并 且该算法会消耗了更多的系统资源，比如创建多个线程和多个双端队列。

# Fork/Join框架的设计

- 分割任务

​		首先我们需要有一个fork类来把大任务分割成子任务，有可能子任务还是很大，所以还需要不停地分割，直到分割出的子任务足够小。

- 执行任务并合并结果

​		分割的子任务分别放在双端队列里，然后几个启动线程分 别从双端队列里获取任务执行。子任务执行完的结果都统一放在一个队列里，启动一个线程 从队列里拿数据，然后合并这些数据。

Fork/Join**使用两个类来完成以上两件事情**

- ForkJoinTask:我们要使用ForkJoin框架，必须首先创建一个ForkJoin任务。它提供在任务 中执行fork()和join()操作的机制。通常情况下，我们不需要直接继承ForkJoinTask类，只需要继承它的子类，Fork/Join框架提供了以下两个子类。
  - RecursiveAction:用于没有返回结果的任务。 
  - RecursiveTask:用于有返回结果的任务。 

- ForkJoinPool:ForkJoinTask需要通过ForkJoinPool来执行。
  - 任务分割出的子任务会添加到当前工作线程所维护的双端队列中，进入队列的头部。当 一个工作线程的队列里暂时没有任务时，它会随机从其他工作线程的队列的尾部获取一个任 务。

**流程图如下**

![image-20220513105942061](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205131059334.png)

# 使用Fork/Join框架

```java
package com.zjmByte.ArtConcurrentBook.chapterSix;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.Future;
import java.util.concurrent.RecursiveTask;

public class CountTask extends RecursiveTask<Long> {


    private static final int THRESHOLD = 200000; // 阈值
    private int              start;
    private int              end;

    public CountTask(int start, int end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        long sum = 0;

        // 如果任务足够小就计算任务
        boolean canCompute = (end - start) <= THRESHOLD;
        if (canCompute) {
            for (int i = start; i <= end; i++) {
                sum += i;
            }
        } else {
            // 如果任务大于阈值，就分裂成两个子任务计算
            int middle = (start + end) / 2;
            CountTask leftTask = new CountTask(start, middle);
            CountTask rightTask = new CountTask(middle + 1, end);
            //执行子任务
            leftTask.fork();
            rightTask.fork();
            //等待子任务执行完，并得到其结果
            long leftResult = leftTask.join();
            long rightResult = rightTask.join();
            //合并子任务
            sum = leftResult + rightResult;
        }
        return sum;
    }

    public static void main(String[] args) {
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        // 生成一个计算任务，负责计算
        CountTask task = new CountTask(1, 400000000);

        // 执行一个任务
        Future<Long> result = forkJoinPool.submit(task);
        //判断是否有异常
        if (task.isCompletedAbnormally()){
            System.out.println(task.getException().getMessage());
        }
        try {
            System.out.println(result.get());
        } catch (InterruptedException | ExecutionException e) {
        }
    }
}

```

​		通过这个例子，我们进一步了解`ForkJoinTask，ForkJoinTask`与一般任务的主要区别在于它 需要实现`compute`方法，在这个方法里，首先需要判断任务是否足够小，如果足够小就直接执行任务。如果不足够小，就必须分割成两个子任务，每个子任务在调用`fork`方法时，又会进入 `compute`方法，看看当前子任务是否需要继续分割成子任务，如果不需要继续分割，则执行当 前子任务并返回结果。使用`join`方法会等待子任务执行完并得到其结果。