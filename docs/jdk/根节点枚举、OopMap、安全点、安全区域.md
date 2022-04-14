### 根节点枚举

​		首先，固定可作为GC Roots的节点主要在全局性的引用(例如常量或类静态属性)与执行上下文(例如 栈帧中的本地变量表)中

​		其次，一般方法区的大小就常有数百上千兆，里面的类、常量等更是恒河沙数，若要逐个检 查以这里为起源的引用肯定得消耗不少时间

​		并且，为了保证结果的准确性，根节点枚举期间，必须暂停用户线程。这样就会出现`stop the world`

**经过上面的介绍，我们怎么进行高效的回收，来看看HotSpot是怎么解决的。**

​		在HotSpot 的解决方案里，是使用一组称为`OopMap`的数据结构来存放对象引用。一旦类加载动作完成的时候， HotSpot就会把对象内什么偏移量上是什么类型的数据计算出来，在即时编译过程中，也会在**特定的位置**记录下栈里和寄存器里哪些位置是引用。

​		接下来我们将会讨论`OopMap`和`特定的位置`。

### 什么是OopMap

​		GC Roots 枚举的过程中，是需要暂停用户线程的，对栈进行扫描，找到哪些地方存储了对象的引用。

​		然而，栈存储的数据不止是对象的引用，因此对整个栈进行全量扫描，显然是很耗费时间，影响性能的。

​		因此，在 HotSpot 中采取了空间换时间的方法，使用 OopMap 来存储栈上的对象引用的信息。

​		在 GC Roots 枚举时，只需要遍历每个栈桢的 OopMap，通过 OopMap 存储的信息，快捷地找到 GC Roots。

OopMap 中存储了两种对象的引用：

> ◉ 栈里和寄存器内的引用
> 在即时编译中，在特定的位置记录下栈里和寄存器里哪些位置是引用
>
> ◉ 对象内的引用
> 类加载动作完成时，HotSpot 就会计算出对象内什么偏移量上是什么类型的数据
> 注：把存储单元的实际地址与其所在段的段地址之间的距离称为段内偏移，也称为有效地址或偏移量，因此，实际地址=所在段的起始地址+偏移量

​		在 JVM中，一个线程为一个栈，一个栈由多个栈桢组成，一个栈桢对应一个方法，一个栈帧可能有多个 OopMap。

假设，这两个方法都只有一个 OopMap，并且是在方法返回之前：

```java
// 方法1存储在栈帧3
public void testMethod1() {
    // 栈里和寄存器内的引用
    DemoD demoD = new DemoD();
}

// 方法2存储在栈帧8
public void testMethod2() {
    // 栈里和寄存器内的引用
    DemoA demoA = new DemoA();
    // 对象内的引用
    demoA.setDemoC(new DemoC());
    
    // 栈里和寄存器内的引用
    DemoA demoB = new DemoB();
} 
```

那么 testMethod1() 和 testMethod2() 的 OopMap 如下图所示：

![img](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204141747120.jpg)图1



​		因此，可以理解为 OopMap 就是商场的商品清单，清单上记录着每一种商品的所在位置和数量，通过清单可以直接到对应的货架上找到商品。

​		如果没有这份清单，需要寻找一件商品的时候，就只能从头开始，按顺序翻找每一个货架上的商品，直到找到对应的商品。

### 什么是Safe Point

​		然而，在程序执行的过程中，对象之间的引用关系随时都会发生改变，这意味着对应的 OopMap 需要同步进行更新。

​		如果每一条指令的执行，都生成（或更新）对应的OopMap，那么将会占用大量的内存空间，增加了 GC 的空间成本。

​		因此，针对这个问题，JVM 引入了 Safe Point 的概念，只有在 Safe Point 才会生成（或更新）对应的 OopMap。

​		Safe Point 就是一个安全点，可以理解为用户线程执行过程中的一些特殊位置。线程执行到 Safe Point 的时候，OopMap 保存了当前线程的上下文，当线程执行到这些位置的时候，说明线程当前的状态是确定的，线程有哪些对象、使用了哪些内存。

那么，哪些地方适合放置 Safe Point？

> ◉ 所有的非计数循环的末尾
> （防止循环体的执行时间太长，一直进入不了 Safe Point）
>
> ◉ 所有方法返回之前
>
> ◉ 每条 Java 编译后的字节码的边界

例如，在方法返回之前插入 Safe Point，那么栈帧8只有一个 OopMap：

![img](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204141748507.jpg)图2

​		除此之外，Safe Point 的数量不能太少，太少会导致进入 Safe Point 的前置时间过长，以至于垃圾回收线程等待的时间太长。

​		Safe Point 的数量也不能太多，太多意味着将会频繁生成（或更新）OopMap ，会有性能损耗。

当所有线程都到达Safe Point，有两种方法中断线程：

> ◉ 抢占式中断(Preemptive Suspension)
> JVM会中断所有线程，然后依次检查每个线程中断的位置是否为Safe Point，如果不是则恢复用户线程，让它执行至 Safe Point 再阻塞。
>
> ◉ 主动式中断(Voluntary Suspension)
> 大部分 JVM 实现都是采用主动式中断，需要阻塞用户线程的时候，首先做一个标志，用户线程会主动轮询这个标志位，如果标志位处于就绪状态，就自行中断。

​		因此，可以理解为 OopMap + Safe Point 就是商店的商品清点，只需要在特定的时间进行，例如每天开始营业之前和结束营业之后。而不是每卖出一件商品，或者每上架一件商品，就进行一次商品清点。

**文章推荐**

[安全点引发的长时间停顿问题](https://mp.weixin.qq.com/s/KDUccdLALWdjNBrFjVR74Q)

### 什么是Safe Region

​		然而，实际情况中 Safe Point 仍然存在缺陷，线程执行的过程中，Safe Point 可以发挥很好的作用。

​		可如果线程没有执行呢？即线程没有分配到 CPU 片，例如：线程处于 Sleep 状态或者 Blocked 状态，那么线程就无法达到 Safe Point。

​		因此，针对这个问题，JVM 引入了 Safe Region 的概念。Safe Region 是一片区域，在这个区域的代码片段，引用关系不会发生变化，因此，在 Safe Region 中任意地方开始垃圾收集都是安全的。

​		可以理解为 Safe Region 就是 Safe Point 的扩展，点动成线。

​		线程执行到 Safe Region 时，首先标记线程已经进入 Safe Region，当线程将要离开 Safe Region 时，线程需要检查 JVM 是否已经完成 GC Roots 枚举。如果尚未完成，则需要一直等待，直到 GC Roots 枚举完成。

整个过程如下图所示：

![img](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204141748602.jpg)图3

例如，下面这个方法的：

```java
// 方法2存储在栈帧8
public void testMethod2() {
    // 栈里和寄存器内的引用
    DemoA demoA = new DemoA();
    // 对象内的引用
    demoA.setDemoC(new DemoC());
    
    // Safe Region 的开始
    System.out.println(demoA);
    System.out.println(demoA.getDemoB());
    // Safe Region 的结束
    // 在此之间对象引用关系并没有发生改变
}
```

​		可以理解为 Safe Region 就是商店休假的整个时间段，在此期间，商品的数量没有发生改变，是确定的。