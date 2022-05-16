[TOC]



# Java多线程基础知识

## 线程简介

### 线程的状态

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

### Daemon线程

​		`Daemon`线程是一种支持型线程，因为它主要被用作程序中后台调度以及支持性工作。这 意味着，当一个Java虚拟机中不存在非`Daemon`线程的时候，Java虚拟机将会退出。可以通过调 用`Thread.setDaemon(true)`将线程设置为`Daemon`线程。

​		`Daemon`线程被用作完成支持性工作，但是在Java虚拟机退出时`Daemon`线程中的`finally`块并不一定会执行，示例如代码清单所示。

```java
public class Daemon {

    public static void main(String[] args) {
        Thread thread = new Thread(new DaemonRunner());
        thread.setDaemon(true);
        thread.start();
    }

    static class DaemonRunner implements Runnable {
        @Override
        public void run() {
            try {
                SleepUtils.second(100);
            } finally {
                System.out.println("DaemonThread finally run.");
            }
        }
    }
}
```

​		运行`Daemon`程序，可以看到在终端或者命令提示符上没有任何输出。`main`线程(非` Daemon`线程)在启动了线程`DaemonRunner`之后随着`main`方法执行完毕而终止，而此时`Java`虚拟 机中已经没有非`Daemon`线程，虚拟机需要退出。`Java`虚拟机中的所有`Daemon`线程都需要立即 终止，因此`DaemonRunner`立即终止，但是`DaemonRunner`中的`finally`块并没有执行。

## 启动和终止线程

### 构造线程

​		在运行线程之前首先要构造一个线程对象，线程对象在构造的时候需要提供线程所需要 的属性，如线程所属的线程组、线程优先级、是否是`Daemon`线程等信息。代码清单所示的 代码摘自`java.lang.Thread`中对线程进行初始化的部分。

```java
private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize, AccessControlContext acc,
                      boolean inheritThreadLocals) {
        if (name == null) {
            throw new NullPointerException("name cannot be null");
        }

        this.name = name;
				//当前线程就是该线程的父线程
        Thread parent = currentThread();
        SecurityManager security = System.getSecurityManager();
        if (g == null) {
            /* Determine if it's an applet or not */

            /* If there is a security manager, ask the security manager
               what to do. */
            if (security != null) {
                g = security.getThreadGroup();
            }

            /* If the security doesn't have a strong opinion of the matter
               use the parent thread group. */
            if (g == null) {
                g = parent.getThreadGroup();
            }
        }

        /* checkAccess regardless of whether or not threadgroup is
           explicitly passed in. */
        g.checkAccess();

        /*
         * Do we have the required permissions?
         */
        if (security != null) {
            if (isCCLOverridden(getClass())) {
                security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
            }
        }

        g.addUnstarted();

        this.group = g;
  			//将daemon、priority属性设置为父线程的对应属性
        this.daemon = parent.isDaemon();
        this.priority = parent.getPriority();
        if (security == null || isCCLOverridden(parent.getClass()))
            this.contextClassLoader = parent.getContextClassLoader();
        else
            this.contextClassLoader = parent.contextClassLoader;
        this.inheritedAccessControlContext =
                acc != null ? acc : AccessController.getContext();
        this.target = target;
        setPriority(priority);
  			//将父线程的InheritableThreadLocal复制过来
        if (inheritThreadLocals && parent.inheritableThreadLocals != null)
            this.inheritableThreadLocals =
                ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
        /* Stash the specified stack size in case the VM cares */
        this.stackSize = stackSize;

        /* Set thread ID 
        	 分配一个线程ID
        */
        tid = nextThreadID();
    }
```

​		在上述过程中，一个新构造的线程对象是由其`parent`线程来进行空间分配的，而`child`线程 继承了`parent`是否为`Daemon`、优先级和加载资源的`contextClassLoader`以及可继承的 `ThreadLocal`，同时还会分配一个唯一的ID来标识这个`child`线程。至此，一个能够运行的线程对象就初始化好了，在堆内存中等待着运行。

### 启动线程

​		线程对象在初始化完成之后，调用`start()`方法就可以启动这个线程。线程`start()`方法的含义 是:当前线程(即`parent`线程)同步告知`Java`虚拟机，只要线程规划器空闲，应立即启动调用 `start()`方法的线程。

> 此处有一个面试题：为什么不直接调用`run`方法去执行一个线程？
>
> 如果直接调用线程的`run`方法，其实不是真正的创建了一个线程去执行`run`方法里面的内容，而是实际调用`run`方法的线程在执行`run`方法，所以实际并没有开启线程。

## 线程间通信

### 等待/通知机制

​		等待/通知的相关方法是任意`Java`对象都具备的，因为这些方法被定义在所有对象的超类` java.lang.Object`上，方法和描述如表所示。

![image-20220510135947653](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205101359689.png)

​		等待/通知机制，是指一个线程A调用了对象O的`wait()`方法进入等待状态，而另一个线程B 调用了对象O的`notify()`或者`notifyAll()`方法，线程A收到通知后从对象O的`wait()`方法返回，进而 执行后续操作。上述两个线程通过对象O来完成交互，而对象上的`wait()`和`notify/notifyAll()`的 关系就如同开关信号一样，用来完成等待方和通知方之间的交互工作。

​		在代码清单所示的例子中，创建了两个线程——`WaitThread`和`NotifyThread`，前者检查 flag值是否为false，如果符合要求，进行后续操作，否则在lock上等待，后者在睡眠了一段时间 后对lock进行通知，示例如下所示。

```java
package com.zjmByte.ArtConcurrentBook.chapterFour;

/**
 *
 */
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.concurrent.TimeUnit;

/**
 * 6-11
 */
public class WaitNotify {
    static boolean flag = true;
    static Object  lock = new Object();

    public static void main(String[] args) throws Exception {
        Thread waitThread = new Thread(new Wait(), "WaitThread");
        waitThread.start();
        TimeUnit.SECONDS.sleep(1);

        Thread notifyThread = new Thread(new Notify(), "NotifyThread");
        notifyThread.start();
    }

    static class Wait implements Runnable {
        public void run() {
            // 加锁，拥有lock的Monitor
            synchronized (lock) {
                //当条件不满足时，继续wait，同时释放了lock的锁
                while (flag) {
                    try {
                        System.out.println(Thread.currentThread() + " flag is true. wait @ "
                                + new SimpleDateFormat("HH:mm:ss").format(new Date()));
                        lock.wait();
                    } catch (InterruptedException e) {
                    }
                }
                // 条件满足时，完成工作
                System.out.println(Thread.currentThread() + " flag is false. running @ "
                        + new SimpleDateFormat("HH:mm:ss").format(new Date()));
            }
        }
    }

    static class Notify implements Runnable {
        public void run() {
            // 加锁，拥有lock的Monitor
            synchronized (lock) {
                // 获取lock的锁，然后进行通知，通知时不会释放lock的锁，
                // 直到当前线程释放了lock后，WaitThread才能从wait方法中返回
                System.out.println(Thread.currentThread() + " hold lock. notify @ " + new SimpleDateFormat("HH:mm:ss").format(new Date()));
                lock.notifyAll();
                flag = false;
                SleepUtils.second(5);
            }
            //再次加锁
            synchronized (lock) {
                System.out.println(Thread.currentThread() + " hold lock again. sleep @ "
                        + new SimpleDateFormat("HH:mm:ss").format(new Date()));
                SleepUtils.second(5);
            }
        }
    }

}
```

​		而上述例子主要说明了调用`wait()、notify()`以 及`notifyAll()`时需要注意的细节，如下。

- 使用wait()、notify()和notifyAll()时需要先对调用对象加锁。 
- 调用wait()方法后，线程状态由RUNNING变为WAITING，并将当前线程放置到对象的等待队列。

- notify()或notifyAll()方法调用后，等待线程依旧不会从wait()返回，需要调用notify()或 notifAll()的线程释放锁之后，**等待线程才有机会从wait()返回**。

- notify()方法将等待队列中的一个等待线程从等待队列中移到同步队列中，而notifyAll() 方法则是将等待队列中所有的线程全部移到同步队列，被移动的线程状态由WAITING变为 BLOCKED。

- **从wait()方法返回的前提是获得了调用对象的锁。**

​		下图描述了上述示例的过程。

![image-20220510140358778](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205101403824.png)

​		在图中，`WaitThread`首先获取了对象的锁，然后调用对象的`wait()`方法，从而放弃了锁 并进入了对象的等待队列`WaitQueue`中，进入等待状态。由于`WaitThread`释放了对象的锁，`NotifyThread`随后获取了对象的锁，并调用对象的`notify()`方法，将`WaitThread从WaitQueue`移到 `SynchronizedQueue`中，此时`WaitThread`的状态变为阻塞状态。`NotifyThread`释放了锁之后， `WaitThread`再次获取到锁并从`wait()`方法返回继续执行。