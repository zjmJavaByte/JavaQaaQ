[TOC]

# 关键字volatile

## volatile的特性

​		理解volatile特性的一个好方法是把对volatile变量的单个读/写，看成是使用同一个锁对这些单个读/写操作做了同步。下面通过具体的示例来说明，示例代码如下。

```java
class VolatileFeaturesExample {
    volatile long vl = 0L; //使用volatile声明64位的long型变量

    public void set(long l) {
        vl = l; //单个volatile变量的写
    }

    public void getAndIncrement() {
        vl++; //复合(多个)volatile变量的读/写
    }

    public long get() {
        return vl; //单个volatile变量的读
    }
}
```

假设有多个线程分别调用上面程序的3个方法，这个程序在语义上和下面程序等价。

```java
class VolatileFeaturesExample1 {
    long vl = 0L; // 64位的long型普通变量

    public synchronized void set(long l) {//对单个的普通变量的写用同一个锁同步
        vl = l;
    }

    public void getAndIncrement() { //普通方法调用
        long temp = get(); //调用已同步的读方法
        temp += 1L; //普通写操作
        set(temp); //调用已同步的写方法
    }

    public synchronized long get() { //对单个的普通变量的读用同一个锁同步
        return vl;
    }
}
```

​		如上面示例程序所示，**一个volatile变量的单个读/写操作，与一个普通变量的读/写操作都是使用同一个锁来同步，它们之间的执行效果相同。**

​		**锁的`happens-before`规则保证释放锁和获取锁的两个线程之间的内存可见性，这意味着对 一个`volatile`变量的读，`总是能看到(任意线程)对这个volatile变量最后的写入。`**

​		**锁的语义决定了临界区代码的执行具有原子性。**这意味着，即使是64位的`long`型和`double `型变量，只要它是`volatile`变量，对该变量的读/写就具有原子性。如果是多个`volatile`操作或类 似于`volatile++`这种复合操作，这些操作整体上不具有原子性。

​		简而言之，`volatile`变量自身具有下列特性。 

- 可见性:对一个`volatile`变量的读，总是能看到(任意线程)对这个`volatile`变量最后的写入。

- 原子性:对任意单个`volatile`变量的读/写具有原子性，但类似于`volatile++`这种复合操作不 具有原子性。

## 可见性

​		当一个变量被定义成`volatile`之后，它将具备两项特性:第一项是保证此变量对所有线程的可见性，这里的“可见性”是指当一条线程修改了这个变量的值，新值对于其他线程来说是可以立即得知的(**如果对声明了volatile的 变量进行写操作，JVM就会向处理器发送一条Lock前缀的指令，将这个变量所在缓存行的数据 写回到系统内存。但是，就算写回到内存，如果其他处理器缓存的值还是旧的，再执行计算操 作就会有问题。所以，在多处理器下，为了保证各个处理器的缓存是一致的，就会实现缓存一致性协议，每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状 态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。**)。而普通变量并不能做到这一点，普通变量的值在线程间传递时均需要通过主内存来完成。比如， 线程`A`修改一个普通变量的值，然后向主内存进行回写(**但是回写操作不知道何时会写到住内存**)，另外一条线程B在线程A回写完成了之后再对主内存进行读取操作，新变量值才会对线程`B`可见。

​		关于`volatile`变量的可见性，经常会被开发人员误解，他们会误以为下面的描述是正确的:“ `volatile` 变量对所有线程是立即可见的，对`volatile`变量所有的写操作都能立刻反映到其他线程之中。换句话 说，`volatile`变量在各个线程中是一致的，所以基于`volatile`变量的运算在并发下是线程安全的”。这句话的论据部分并没有错，但是由其论据并不能得出“ 基于`volatile`变量的运算在并发下是线程安全的”这样 的结论。`volatile`变量在各个线程的工作内存中是不存在一致性问题的(从物理存储的角度看，各个线 程的工作内存中`volatile`变量也可以存在不一致的情况，但由于每次使用之前都要先刷新，执行引擎看不到不一致的情况，因此可以认为不存在一致性问题)，但是Java里面的运算操作符并非原子操作， 这导致`volatile`变量的运算在并发下一样是不安全的，我们可以通过一段简单的演示来说明原因，请看以下代码清中演示的例子。

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

## volatile写-读建立的happens-before关系

​		从内存语义的角度来说，`volatile`的写-读与锁的释放-获取有相同的内存效果:`volatile`写和 锁的释放有相同的内存语义;`volatile`读与锁的获取有相同的内存语义。

```java
class VolatileExample {
    int a = 0;
    volatile boolean flag = false;

    public void writer() {
        a = 1; //1
        flag = true; //2
    }

    public void reader() {
        if (flag) { //3
            int i = a; //4
            //......
        }
    }
}
```

​		假设线程A执行`writer()`方法之后，线程B执行`reader()`方法。根据`happens-before`规则，这个过程建立的`happens-before`关系可以分为3类:

1)根据程序次序规则(一个线程内保证语义的串行性)，`1 happens-before 2;3 happens-before 4`。 

2)根据`volatile`规则(volatile`变量的写先发生于读，这保证了`volatile`变量的可见性)，`2 happens-before 3`。 

3)根据`happens-before`的传递性规则(A先于B，B先于C，那么A必然先于C)，`1 happens-before 4`。 

上述`happens-before`关系的图形化表现形式如下。

![image-20220508190215145](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205081902838.png)

​		在上图中，每一个箭头链接的两个节点，代表了一个`happens-before`关系。黑色箭头表示程 序顺序规则;橙色箭头表示`volatile`规则;蓝色箭头表示组合这些规则后提供的`happens-before`保 证。

## volatile写-读的内存语义

**volatile写的内存语义如下**

当写一个`volatile`变量时，`JMM`会把该线程对应的本地内存中的共享变量值刷新到主内存。

**volatile读的内存语义如下**

当读一个`volatile`变量时，JMM会把该线程对应的本地内存置为无效。线程接下来将从主 内存中读取共享变量。

## volatile内存语义的实现

​		重排序（重排序是指编译器和处理器为了优化程序性能而对指令序列进行重新排序的一种手段）分为编译器重排序和处理器重排序。为了实现volatile内存语义，JMM 会分别限制这两种类型的重排序类型。

**volatile重排序规则表**

![image-20220508215315332](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205082153376.png)

- 第三行最后一个单元格的意思是:在程序中，当第一个操作为普通变量的读或 写时，如果第二个操作为`volatile`写，则编译器不能重排序这两个操作。

- 当第二个操作是`volatile`写时，不管第一个操作是什么，都不能重排序。这个规则确保`volatile`写之前的操作不会被编译器重排序到`volatile`写之后。 
- 当第一个操作是`volatile`读时，不管第二个操作是什么，都不能重排序。这个规则确保`volatile`读之后的操作不会被编译器重排序到`volatile`读之前。

- 当第一个操作是`volatile`写，第二个操作是`volatile`读时，不能重排序。

​		为了实现`volatile`的内存语义，编译器在生成字节码时，会在指令序列中插入内存屏障来 禁止特定类型的处理器重排序。对于编译器来说，发现一个最优布置来最小化插入屏障的总 数几乎不可能。为此，`JMM`采取保守策略。下面是基于保守策略的`JMM`内存屏障插入策略:

- 在每个volatile写操作的前面插入一个StoreStore屏障。

- 在每个volatile写操作的后面插入一个StoreLoad屏障。

- 在每个volatile读操作的后面插入一个LoadLoad屏障。

- 在每个volatile读操作的后面插入一个LoadStore屏障。

​		上述内存屏障插入策略非常保守，但它可以保证在任意处理器平台，任意的程序中都能 得到正确的`volatile`内存语义。

​		下面是保守策略下，`volatile`写插入内存屏障后生成的指令序列示意图，如图所示。

![image-20220508220736188](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205082207230.png)

​		图中的`StoreStore`屏障可以保证在volatile写之前，其前面的所有普通写操作已经对任 意处理器可见了。这是因为`StoreStore`屏障将保障上面所有的普通写在`volatile`写之前刷新到主内存。

​		这里比较有意思的是，`volatile`写后面的`StoreLoad`屏障。此屏障的作用是避免`volatile`写与后面可能有的volatile读/写操作重排序。因为编译器常常无法准确判断在一个`volatile`写的后面 是否需要插入一个`StoreLoad`屏障(比如，一个`volatile`写之后方法立即`return`)。为了保证能正确 实现`volatile`的内存语义，JMM在采取了保守策略:在每个volatile写的后面，或者在每个volatile 读的前面插入一个`StoreLoad`屏障。从整体执行效率的角度考虑，`JMM`最终选择了在每个 `volatile`写的后面插入一个`StoreLoad`屏障。因为`volatile`写-读内存语义的常见使用模式是:一个 写线程写`volatile变量，多个读线程读同一个`volatile`变量。当读线程的数量大大超过写线程时， 选择在`volatile`写之后插入`StoreLoad`屏障将带来可观的执行效率的提升。从这里可以看到`JMM `在实现上的一个特点:首先确保正确性，然后再去追求执行效率。

​		下面是在保守策略下，`volatile`读插入内存屏障后生成的指令序列示意图，如图所示。

![image-20220508220814114](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205082208158.png)

​		图的`LoadLoad`屏障用来禁止处理器把上面的`volatile`读与下面的普通读重排序。 `LoadStore`屏障用来禁止处理器把上面的`volatil`e读与下面的普通写重排序。

​		上述`volatile`写和`volatile`读的内存屏障插入策略非常保守。在实际执行时，只要不改变` volatile`写-读的内存语义，编译器可以根据具体情况省略不必要的屏障。下面通过具体的示例 代码进行说明。

```java
class VolatileBarrierExample {
    int          a;
    volatile int v1 = 1;
    volatile int v2 = 2;

    void readAndWrite() {
        int i = v1; //第一个volatile读
        int j = v2; // 第二个volatile读
        a = i + j; //普通写
        v1 = i + 1; // 第一个 volatile写
        v2 = j * 2; //第二个 volatile写
    }

    //....                //其他方法
}
```

针对`readAndWrite()`方法，编译器在生成字节码时可以做如下的优化。

![image-20220508221157065](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205082211112.png)

​		注意，最后的`StoreLoad`屏障不能省略。因为第二个`volatile`写之后，方法立即`return`。此时编 译器可能无法准确断定后面是否会有`volatile`读或写，为了安全起见，编译器通常会在这里插 入一个`StoreLoad`屏障。

​		上面的优化针对任意处理器平台，由于不同的处理器有不同“松紧度”的处理器内存模 型，内存屏障的插入还可以根据具体的处理器内存模型继续优化。以`X86`处理器为例，上图中除最后的`StoreLoad`屏障外，其他的屏障都会被省略。

​		前面保守策略下的`volatile`读和写，在X86处理器平台可以优化成下图所示。

​		前文提到过，`X86`处理器仅会对写-读操作做重排序。`X86`不会对读-读、读-写和写-写操作 做重排序，因此在`X86`处理器中会省略掉这3种操作类型对应的内存屏障。在`X86`中，`JMM`仅需 在`volatile`写后面插入一个`StoreLoad`屏障即可正确实现`volatile`写-读的内存语义。这意味着在 `X86`处理器中，`volatile`写的开销比`volatile`读的开销会大很多(因为执行`StoreLoad`屏障开销会比 较大)。

![image-20220508221342773](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205082213812.png)

[如何正确使用volatile](https://www.cnblogs.com/davidwang456/p/6110820.html)