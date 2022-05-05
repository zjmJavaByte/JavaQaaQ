# 关键字volatile

## 可见性

​		当一个变量被定义成`volatile`之后，它将具备两项特性:第一项是保证此变量对所有线程的可见 性，这里的“可见性”是指当一条线程修改了这个变量的值，新值对于其他线程来说是可以立即得知 的。而普通变量并不能做到这一点，普通变量的值在线程间传递时均需要通过主内存来完成。比如， 线程`A`修改一个普通变量的值，然后向主内存进行回写，另外一条线程B在线程A回写完成了之后再对 主内存进行读取操作，新变量值才会对线程`B`可见。

​		关于`volatile`变量的可见性，经常会被开发人员误解，他们会误以为下面的描述是正确的:“ `volatile` 变量对所有线程是立即可见的，对`volatile`变量所有的写操作都能立刻反映到其他线程之中。换句话 说，`volatile`变量在各个线程中是一致的，所以基于`volatile`变量的运算在并发下是线程安全的”。这句话 的论据部分并没有错，但是由其论据并不能得出“ 基于`volatile`变量的运算在并发下是线程安全的”这样 的结论。`volatile`变量在各个线程的工作内存中是不存在一致性问题的(从物理存储的角度看，各个线 程的工作内存中`volatile`变量也可以存在不一致的情况，但由于每次使用之前都要先刷新，执行引擎看 不到不一致的情况，因此可以认为不存在一致性问题)，但是Java里面的运算操作符并非原子操作， 这导致`volatile`变量的运算在并发下一样是不安全的，我们可以通过一段简单的演示来说明原因，请看以下代码清中演示的例子。

```java
package org.fenixsoft.jvm.chapter12;

/**
 * volatile变量自增运算测试
 *
 * @author zzm
 */
public class VolatileTest {

    public static volatile int race = 0;

    public static void increase() {
        race++;
    }

    private static final int THREADS_COUNT = 20;

    public static void main(String[] args) {
        Thread[] threads = new Thread[THREADS_COUNT];
        for (int i = 0; i < THREADS_COUNT; i++) {
            threads[i] = new Thread(new Runnable() {
                @Override
                public void run() {
                    for (int i = 0; i < 10000; i++) {
                        increase();
                    }
                }
            });
            threads[i].start();
        }

        // 等待所有累加线程都结束
        while (Thread.activeCount() > 1)
            Thread.yield();

        System.out.println(race);
    }
}
```

> 使用IntelliJ IDEA的读者请注意，在IDEA中运行这段程序，会由于IDE自动创建一条名为M onitor Ct rl-Break的线程(从名字看应该是监控Ct rl-Break中断信号的)而导致w hile循环无法结束，改为大于2 或者用T hread::join()方法代替可以解决该问题。		

​		这段代码发起了20个线程，每个线程对race变量进行10000次自增操作，如果这段代码能够正确并 发的话，最后输出的结果应该是200000。读者运行完这段代码之后，并不会获得期望的结果，而且会 发现每次运行程序，输出的结果都不一样，都是一个小于200000的数字。这是为什么呢?

​		问题就出在自增运算`race++`之中，我们用Javap反编译这段代码后会得到如下代码：

```java
 public static void increase();
    Code:
       0: getstatic     #2                  // Field race:I
       3: iconst_1
       4: iadd
       5: putstatic     #2                  // Field race:I
       8: return
```

​		发现只有一行代码的`increase()`方法在Class文件中是由4条字节码指令构成(`return`指令不是由`race++`产生 的，这条指令可以不计算)，从字节码层面上已经很容易分析出并发失败的原因了:当`getstatic`指令把 `race`的值取到操作栈顶时，`volatile`关键字保证了`race`的值在此时是正确的，但是在执行`iconst _1、iadd`这些指令的时候，其他线程可能已经把`race`的值改变了，而操作栈顶的值就变成了过期的数据，所以 `putstatic`指令执行后就可能把较小的`race`值同步回主内存之中。

​		实事求是地说，笔者使用字节码来分析并发问题仍然是不严谨的，因为即使编译出来只有一条字 节码指令，也并不意味执行这条指令就是一个原子操作。一条字节码指令在解释执行时，解释器要运 行许多行代码才能实现它的语义。如果是编译执行，一条字节码指令也可能转化成若干条本地机器码 指令。此处使用`-XX:+PrintAssembly`参数输出反汇编来分析才会更加严谨一些，但是考虑到读者阅读 的方便性，并且字节码已经能很好地说明问题，所以此处使用字节码来解释。

​		由于`volatile`变量只能保证可见性，在不符合以下两条规则的运算场景中，我们仍然要通过加锁 (使用`synchronized、java.util.concurrent `中的锁或原子类)来保证原子性:

- 运算结果并不依赖变量的当前值，或者能够确保只有单一的线程修改变量的值。

- 变量不需要与其他的状态变量共同参与不变约束。

​		而在像如下代码清单所示的这类场景中就很适合使用`volatile`变量来控制并发，当`shutdown()`方法被 调用时，能保证所有线程中执行的`doWork()`方法都立即停下来。

```java
volatile boolean shutdownRequested;

    public void shutdown() {
        shutdownRequested = true;
    }

    public void doWork() {
        while (!shutdownRequested) {
		// 代码的业务逻辑 
        }
     }
```

## 禁止指令重排序优化

​		使用`volatile`变量的第二个语义是禁止指令重排序优化，普通的变量仅会保证在该方法的执行过程 中所有依赖赋值结果的地方都能获取到正确的结果，而不能保证变量赋值操作的顺序与程序代码中的 执行顺序一致。因为在同一个线程的方法执行过程中无法感知到这点，这就是Java内存模型中描述的 所谓“线程内表现为串行的语义”(`Within-Thread As-If-Serial Semantics`)。

上面描述仍然比较拗口难明，我们还是继续通过一个例子来看看为何指令重排序会干扰程序的并 发执行。演示程序如代码清单所示:

```java
				Map configOptions;
        char[] configText;
// 此变量必须定义为volatile
        volatile boolean initialized = false;
// 假设以下代码在线程A中执行
// 模拟读取配置信息，当读取完成后
// 将initialized设置为true,通知其他线程配置可用 configOptions = new HashMap();
        configText = readConfigFile(fileName);
        processConfigOptions(configText, configOptions);
        initialized = true;
// 假设以下代码在线程B中执行
// 等待initialized为true，代表线程A已经把配置信息初始化完成 
        while (!initialized) {
            sleep();
        }
// 使用线程A中初始化好的配置信息 
        doSomethingWithConfig();
```

​		代码清单中所示的程序是一段伪代码，其中描述的场景是开发中常见配置读取过程，只是我 们在处理配置文件时一般不会出现并发，所以没有察觉这会有问题。读者试想一下，如果定义`initialized`变量时没有使用`volatile`修饰，就可能会由于指令重排序的优化，导致位于线程A中最后一条 代码 `initialized=t rue`被提前执行(这里虽然使用Java作为伪代码，但所指的重排序优化是机器级的优 化操作，提前执行是指这条语句对应的汇编代码被提前执行)，这样在线程B中使用配置信息的代码就可能出现错误，而`volatile`关键字则可以避免此类情况的发生。 指令重排序是并发编程中最容易导致开发人员产生疑惑的地方之一，除了上面伪代码的例子之外，笔者再举一个可以实际操作运行的例子来分析volat ile关键字是如何禁止指令重排序优化的。代码清单所示是一段标准的双锁检测(`Double Check Lock，DCL`)单例[代码，可以观察加入`volatile` 和未加入`volatile`关键字时所生成的汇编代码的差别

```java
public class Singleton {
        private volatile static Singleton instance;

        public static Singleton getInstance() {
            if (instance == null) {
                synchronized (Singleton.class) {
                    if (instance == null) {
                        instance = new Singleton();
                    }
                }
            }
            return instance;
        }

        public static void main(String[] args) {
            Singleton.getInstance();
        }
    }
```

编译后，这段代码对`instance`变量赋值的部分如代码清单所示:

![image-20220505155915582](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205051559621.png)

​		通过对比发现，关键变化在于有`volatile`修饰的变量，赋值后(前面`mov%eax，0x150(%esi)`这句便 是赋值操作)多执行了一个`lock addl$0x0，(%esp)`操作，这个操作的作用相当于一个内存屏障 (`Memory Barrier或Memory Fence`，指重排序时不能把后面的指令重排序到内存屏障之前的位置，注 意不要与第3章中介绍的垃圾收集器用于捕获变量访问的内存屏障互相混淆)，只有一个处理器访问内 存时，并不需要内存屏障;但如果有两个或更多处理器访问同一块内存，且其中有一个在观测另一 个，就需要内存屏障来保证一致性了。

​		这句指令中的`addl$0x0，(%esp )`(把ESP寄存器的值加0)显然是一个空操作，之所以用这个空 操作而不是空操作专用指令`nop`，是因为IA32手册规定lock前缀不允许配合nop指令使用。这里的关键 在于`lock`前缀，查询IA32手册可知，它的作用是将本处理器的缓存写入了内存，该写入动作也会引起 别的处理器或者别的内核无效化`(Invalidate)`其缓存，这种操作相当于对缓存中的变量做了一次前面介绍Java内存模式中所说的`store和write`操作。所以通过这样一个空操作，可让前面volatile变量的修改对其他处理器立即可见。

​		那为何说它禁止指令重排序呢?从硬件架构上讲，指令重排序是指处理器采用了允许将多条指令 不按程序规定的顺序分开发送给各个相应的电路单元进行处理。但并不是说指令任意重排，处理器必 须能正确处理指令依赖情况保障程序能得出正确的执行结果。譬如指令1把地址A中的值加10，指令2 把地址A中的值乘以2，指令3把地址B中的值减去3，这时指令1和指令2是有依赖的，它们之间的顺序 不能重排——`(A+10)*2与A*2+10`显然不相等，但指令3可以重排到指令1、2之前或者中间，只要保证 处理器执行后面依赖到A、B值的操作时能获取正确的A和B值即可。所以在同一个处理器中，重排序 过的代码看起来依然是有序的。因此，`lockaddl$0x0，(%esp)`指令把修改同步到内存时，意味着所有之 前的操作都已经执行完成，这样便形成了“指令重排序无法越过内存屏障”的效果。