# Java多线程基础知识

## 线程的状态

![image-20220509214456276](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205092144316.png)

​		下面我们使用`jstack工`具，尝试查看示例代码运行时的线程信息，更加深入地理解线程状态，示例如代码清单所示

```java
package com.zjmByte.ArtConcurrentBook.chapterFour;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 6-3
 */
public class ThreadState {

    private static Lock lock = new ReentrantLock();

    public static void main(String[] args) {
        new Thread(new TimeWaiting(), "TimeWaitingThread").start();
        new Thread(new Waiting(), "WaitingThread").start();
        // Ê¹ÓÃÁ½¸öBlockedÏß³Ì£¬Ò»¸ö»ñÈ¡Ëø³É¹¦£¬ÁíÒ»¸ö±»×èÈû
        new Thread(new Blocked(), "BlockedThread-1").start();
        new Thread(new Blocked(), "BlockedThread-2").start();
        new Thread(new Sync(), "SyncThread-1").start();
        new Thread(new Sync(), "SyncThread-2").start();
    }

    /**
     * 该线程不断地进行睡眠
     */
    static class TimeWaiting implements Runnable {
        @Override
        public void run() {
            while (true) {
                SleepUtils.second(100);
            }
        }
    }

    /**
     * 该线程在Waiting.class实例上等待
     */
    static class Waiting implements Runnable {
        @Override
        public void run() {
            while (true) {
                synchronized (Waiting.class) {
                    try {
                        Waiting.class.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }

    /**
     * ¸该线程在Blocked.class实例上加锁后，不会释放该锁
     */
    static class Blocked implements Runnable {
        public void run() {
            synchronized (Blocked.class) {
                while (true) {
                    SleepUtils.second(100);
                }
            }
        }
    }

    static class Sync implements Runnable {

        @Override
        public void run() {
            lock.lock();
            try {
                SleepUtils.second(100);
            } finally {
                lock.unlock();
            }

        }

    }
}

public class SleepUtils {
    public static final void second(long seconds) {
        try {
            TimeUnit.SECONDS.sleep(seconds);
        } catch (InterruptedException e) {
        }
    }
}
```

**运行结果**

> "SyncThread-2" #16 prio=5 os_prio=31 tid=0x000000012d849000 nid=0xa503 waiting on condition [0x000000016e536000]
>    java.lang.Thread.State: TIMED_WAITING (sleeping)
>         at java.lang.Thread.sleep(Native Method)
>         at java.lang.Thread.sleep(Thread.java:340)
>         at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386)
>         at com.zjmByte.ArtConcurrentBook.chapterFour.SleepUtils.second(SleepUtils.java:11)
>         at com.zjmByte.ArtConcurrentBook.chapterFour.ThreadState$Sync.run(ThreadState.java:72)
>         at java.lang.Thread.run(Thread.java:748)
>
> "SyncThread-1" #15 prio=5 os_prio=31 tid=0x000000012d848000 nid=0x5d03 waiting on condition [0x000000016e32a000]
>    java.lang.Thread.State: WAITING (parking)
>         at sun.misc.Unsafe.park(Native Method)
>         - parking to wait for  <0x000000076ad036b0> (a java.util.concurrent.locks.ReentrantLock$NonfairSync)
>         at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
>         at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInterrupt(AbstractQueuedSynchronizer.java:837)
>         at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireQueued(AbstractQueuedSynchronizer.java:872)
>         at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquire(AbstractQueuedSynchronizer.java:1202)
>         at java.util.concurrent.locks.ReentrantLock$NonfairSync.lock(ReentrantLock.java:213)
>         at java.util.concurrent.locks.ReentrantLock.lock(ReentrantLock.java:290)
>         at com.zjmByte.ArtConcurrentBook.chapterFour.ThreadState$Sync.run(ThreadState.java:70)
>         at java.lang.Thread.run(Thread.java:748)
>
> "BlockedThread-2" #14 prio=5 os_prio=31 tid=0x0000000158015000 nid=0x5b03 waiting for monitor entry [0x000000016e11e000]
>    java.lang.Thread.State: BLOCKED (on object monitor)
>         at com.zjmByte.ArtConcurrentBook.chapterFour.ThreadState$Blocked.run(ThreadState.java:60)
>         - waiting to lock <0x000000076ad0f028> (a java.lang.Class for com.zjmByte.ArtConcurrentBook.chapterFour.ThreadState$Blocked)
>         at java.lang.Thread.run(Thread.java:748)
>
> "BlockedThread-1" #13 prio=5 os_prio=31 tid=0x000000014f865000 nid=0x5903 waiting on condition [0x000000016df12000]
>    java.lang.Thread.State: TIMED_WAITING (sleeping)
>         at java.lang.Thread.sleep(Native Method)
>         at java.lang.Thread.sleep(Thread.java:340)
>         at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386)
>         at com.zjmByte.ArtConcurrentBook.chapterFour.SleepUtils.second(SleepUtils.java:11)
>         at com.zjmByte.ArtConcurrentBook.chapterFour.ThreadState$Blocked.run(ThreadState.java:60)
>         - locked <0x000000076ad0f028> (a java.lang.Class for com.zjmByte.ArtConcurrentBook.chapterFour.ThreadState$Blocked)
>         at java.lang.Thread.run(Thread.java:748)
>
> "WaitingThread" #12 prio=5 os_prio=31 tid=0x000000014d8ae800 nid=0x5803 in Object.wait() [0x000000016dd06000]
>    java.lang.Thread.State: WAITING (on object monitor)
>         at java.lang.Object.wait(Native Method)
>         - waiting on <0x000000076ad0b140> (a java.lang.Class for com.zjmByte.ArtConcurrentBook.chapterFour.ThreadState$Waiting)
>         at java.lang.Object.wait(Object.java:502)
>         at com.zjmByte.ArtConcurrentBook.chapterFour.ThreadState$Waiting.run(ThreadState.java:44)
>         - locked <0x000000076ad0b140> (a java.lang.Class for com.zjmByte.ArtConcurrentBook.chapterFour.ThreadState$Waiting)
>         at java.lang.Thread.run(Thread.java:748)
>
> "TimeWaitingThread" #11 prio=5 os_prio=31 tid=0x000000014e066800 nid=0x5703 waiting on condition [0x000000016dafa000]
>    java.lang.Thread.State: TIMED_WAITING (sleeping)
>         at java.lang.Thread.sleep(Native Method)
>         at java.lang.Thread.sleep(Thread.java:340)
>         at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386)
>         at com.zjmByte.ArtConcurrentBook.chapterFour.SleepUtils.second(SleepUtils.java:11)
>         at com.zjmByte.ArtConcurrentBook.chapterFour.ThreadState$TimeWaiting.run(ThreadState.java:30)
>         at java.lang.Thread.run(Thread.java:748)

**线程格式**

以`"TimeWaitingThread" #11 prio=5 os_prio=31 tid=0x000000014e066800 nid=0x5703 waiting on condition [0x000000016dafa000]为列`

> 
>
> - "TimeWaitingThread"为线程名称，**在平时创建线程或线程池时请务必取一个见明之义的线程名称，方便排查问题**；
> - prio=5：线程优先级，不用关心
> - tid=0x000000014e066800：线程id，不用关心
> - nid=0x5703：**操作系统映射的线程id**
> - waiting on condition：表示线程正在等待条件状态
> - 0x000000016dafa000：线程栈起始地址

## Daemon线程