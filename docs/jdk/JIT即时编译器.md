[TOC]

## JIT即时编译器

### 即时编译器的作用

​		可以看到`JAVA解释器`和`JIT即时编译器`是平行的.那么有了解释器就可以将.class转换为机器码为什么还需要即时编译器呢?当操作系统执行.class文件时,解释器需要逐行将.class代码解释成机器语言,这个过程会很快,但若相同的代码被多次执行,例如`一个方法被多线程调用,或者代码存在于for循环中,此时在逐行解释就会重复性的执行相同工作.所以出现了即时编译.通过将热点代码一次编译成机器码，存储在方法区`.大概流程如下图

![](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204121542725.png)

### 有哪些编译器

JVM 集成的编译器有两种模式：

> ◉ Client Compiler
>
> ◉ Server Compiler

#### Client Compiler

注重启动速度和局部优化，HotSpot VM 使用的是 Client Compiler C1编译器，简称 C1编译器。

#### Server Compiler

注重全局优化，运行过程中性能更好，由于进行更多的全局分析，所以启动速度会变慢。Hotspot VM 使用有两种：C2编译器（默认）和 Graal编译器（暂时不讨论）。

### 分层编译

C1、C2 都有各自的优缺点，为了综合两者的优势，在编译速度和执行效率之间取得平衡，JVM 引入了一种策略：分层编译。

> ◉ 0级（解释执行）：采用解释执行，不开启性能监控功能（profiling）。
>
> ◉ 1级（简单的 C1 编译）：采用 C1编译器，进行简单可靠的优化，不开启性能监控功能。
>
> ◉ 2级（有限的 C1 编译）：采用 C1编译器，进行更多的优化编译，开启性能监控功能，统计方法计数器的值，以及回边计数器的值等。
>
> ◉ 3级（完全 C1 编译）：完全使用 C1编译器 的所有功能，完全开启性能监控功能。
>
> ◉ 4级（C2 编译）：完全使用 C2编译器 进行编译，进行完全的优化。
>
> 注意：分层编译只能在 Server Compiler 模式下启用。

其中，这几个层次的执行效率是这样的：4 > 1 > 2 > 3 > 0。

C2 代码的执行效率比 C1 的高出30%以上。

1 > 2 > 3 的主要原因是性能监控功能开启越多，其性能开销也越大。

列举几种常见的编译路径，如下图所示：

![](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204121543869.jpg)

### 什么是热点代码

JVM 设置一个阈值，当方法或者代码块的在一定时间内的调用次数超过这个阈值，就会被编译，存入 CodeCache。

再次执行这段代码时，直接从 CodeCache 中读取机器码执行，使执行效率大幅提升。

Java 代码的执行过程如下图所示：

![](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204121545035.jpg)

标记热点代码的行为，叫热点探测，通常有两种方法：

> ◉ 基于计数器的热点探测
>
> ◉ 基于采样的热点探测
>
> ◉ 基于踪迹的热点探测

目前 HotSpot VM 所采用的热点探测方式是基于计数器的热点探测。

### 基于计数器的热点探测

JVM 为每个方法（或者代码块）建立计数器，执行次数超过阈值就会标记为热点代码。

计数器分为两类：方法调用计数器、回边计数器。

**在 Client 模式的前提下：（因为 Server 模式下比较复杂，尤其是启用分层模式的情况）**

**方法调用计数器**

统计方法调用的次数，通过参数 -XX:CompileThreshold 设定阈值。

> 注：分层编译的情况下，参数将会失效

默认情况，方法调用计数器统计的并不是方法调用的次数，而是方法调用的频率，即一定时间内方法调用的次数。

如果，一定的时间内，如果两个计数器之和没达到阈值，那么该方法调用计数器的值将会被减半，这个行为称为方法调用计数器热度的衰减，这段时间称为此方法的统计半衰周期。这个行为是在 JVM 进行垃圾回收时进行的，通过参数 -XX:CounterHalfLifeTime 设置半衰周期，单位秒。

除此之外，通过参数 -XX:-UseCounterDecay 可以关闭热度的衰减，那么方法调用计数器统计的就是方法调用的实际次数。



**回边计数器**

统计循环体代码执行的次数，通常使用 -XX:OnStackReplacePercentage 间接设定阈值。

回边计数器没有热度的衰减，设定的阈值就是方法循环执行的实际次数。

-XX:OnStackReplacePercentage，又称 ORS比率，计算公式如下：

> ◉ Server Compiler
> 方法调用计数器阈值 × OSR比率 / 1000，其中 OSR比默认值933。
> 默认情况下，Client 模式下回边计数器的阈值应该是13995。
>
> ◉ Server Compiler
> 方法调用计数器阈值 × (OSR比率 - 解释器监控比率) / 100，其中 OSR比率 默认140，解释器监控比率默认33。
> 默认情况下，Server 模式下回边计数器阈值应该是10700。

### JIT编译优化

#### 公共子表达式的消除

​		公共子表达式消除是一个普遍应用于各种编译器的经典优化技术，他的含义是：**如果一个表达式`E`已经计算过了，并且从先前的计算到现在E中所有变量的值都没有发生变化，那么`E`的这次出现就成为了公共子表达式**。对于这种表达式，没有必要花时间再对他进行计算，只需要直接用前面计算过的表达式结果代替`E`就可以了。

​		**如果这种优化仅限于程序的基本块内，便称为局部公共子表达式消除（Local Common Subexpression Elimination）如果这种优化范围涵盖了多个基本块，那就称为全局公共子表达式消除（Global CommonSubexpression Elimination**）。举个简单的例子来说明他的优化过程，假设存在如下代码：

```java
int d = (c*b)*12+a+(a+b*c); 
```

如果这段代码交给`Javac`编译器则不会进行任何优化，那生成的代码如下所示，是完全遵照`Java`源码的写法直译而成的(`通过javap -v .class查看`)。 

```java
 public void test(int, int, int);
    descriptor: (III)V
    flags: ACC_PUBLIC
    Code:
      stack=4, locals=5, args_size=4
         0: iload_3			//c
         1: iload_2			//b
         2: imul				//c * b = result1
         3: bipush        12
         5: imul				// result1 * 12 = result12
         6: iload_1			//a
         7: iadd				//result12 + a
         8: iload_1			//a
         9: iload_2			//b
        10: iload_3			//c
        11: imul				//b * c = result13
        12: iadd				//a + result13 = result14
        13: iadd				//result12 + result14 = result15
        14: istore        4
        16: return
      LineNumberTable:
        line 6: 0
        line 7: 16
}
```

​		当这段代码进入到虚拟机即时编译器后，他将进行如下优化：**编译器检测到”cb“与”bc“是一样的表达式，而且在计算期间b与c的值是不变的**。因此，这条表达式就可能被视为： 

```java
int d = E*12+a+(a+E);
```

这时，编译器还可能（取决于哪种虚拟机的编译器以及具体的上下文而定）进行另外一种优化：代数化简（Algebraic Simplification），把表达式变为： 

```java
int d = E*13+a*2; 
```

表达式进行变换之后，再计算起来就可以节省一些时间了。

#### 方法内联

​		一个方法要被执行,首先要从方法区找到类的元数据,在找到Method对象,将该对象压栈到栈顶(**此处的栈就是Java虚拟机栈**),该方法执行完毕后会弹栈.如下图

![img](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204121607224.png)

​		**如果方法的调用层次很深,而方法本身并不复杂则会出现频繁的压栈弹栈,是没必要的。在使用JIT进行即时编译时，将方法调用直接使用方法体中的代码进行替换，这就是方法内联，减少了方法调用过程中压栈与入栈的开销**。同时为之后的一些优化手段提供条件。如果JVM监测到一些小方法被频繁的执行，它会把方法的调用替换成方法体本身。 

比如说下面这个： 

```java
private int add4(int x1， int x2， int x3， int x4) {
　　return add2(x1， x2) + add2(x3， x4);
}
private int add2(int x1， int x2) {
　　return x1 + x2;
} 
```

可以肯定的是运行一段时间后JVM会把add2方法去掉，并把你的代码翻译成： 

```java
private int add4(int x1， int x2， int x3， int x4) {
　　return x1 + x2 + x3 + x4;
} 
```

#### 逃逸分析

- **逃逸分析是 Java 虚拟机中的一种优化技术，但它并不是直接优化代码，而是为其他优化手段提供优化依据的分析技术。**
- **逃逸分析的基本行为就是分析对象动态作用域，当一个对象在方法中被定义后，它可能被外部方法所引用，称为方法逃逸。**
- **可能被外部线程访问到，例如赋值给类变量或可以在其他线程中访问的实例变量，称为线程逃逸。**

对象的三种逃逸状态

- GlobalEscape（全局逃逸） 一个对象的引用逃出了方法或者线程。例如，一个对象的引用是复制给了一个类变量，或者存储在在一个已经逃逸的对象当中，或者这个对象的引用作为方法的返回值返回给了调用方法。
- ArgEscape（参数逃逸） 在方法调用过程中传递对象的引用给调用方法，这种状态可以通过分析被调方法的二进制代码确定。
- NoEscape（没有逃逸） 一个可以进行标量替换的对象，可以不将这种对象分配在堆上。

```java
private Object o;
 
/**
 * 给全局变量赋值，发生逃逸（GlobalEscape）
 */
public void globalVariablePointerEscape() {
    o = new Object();
}
 
/**
 * 方法返回值，发生逃逸（GlobalEscape）
 */
public Object methodPointerEscape() {
    return new Object();
}
 
/**
 * 实例引用传递，发生逃逸（ArgEscape）
 */
public void instancePassPointerEscape() {
    Object o = methodPointerEscape();
}
 
/**
 * 没有发生逃逸（NoEscape）
 */
public void noEscape() {
    Object o = new Object();
}
```

配置逃逸分析

- 开启逃逸分析，对象没有分配在堆上，没有进行GC，而是把对象分配在栈上。 （`-XX:+DoEscapeAnalysis` 开启逃逸分析（JDK8 默认开启，其它版本未测试） ）
- 关闭逃逸分析，对象全部分配在堆上，当堆中对象存满后，进行多次GC，导致执行时间大大延长。堆上分配比栈上分配慢上百倍。（`-XX:-DoEscapeAnalysis` 关闭逃逸分析）
- 可以通过 `-XX:+PrintEscapeAnalysis` 查看逃逸分析的筛选结果。

而对于没有逃逸的对象会进行如下三种优化。

##### 对象的栈内存分配

​		我们知道，在一般情况下，对象和数组元素的内存分配是在堆内存上进行的。但是随着JIT编译器的日渐成熟，很多优化使这种分配策略并不绝对。JIT编译器就可以在编译期间根据逃逸分析的结果，来决定是否可以将对象的内存分配从堆转化为栈。以下一个简单的代码示例

```java
package interview;

public class Obj {

    public static void main(String[] args) {
        for (int i = 0; i < 1000000; i++) {
            alloc();
        }
        // 为了方便查看堆内存中对象个数，线程sleep
        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e1) {
            e1.printStackTrace();
        }
    }

    private static void alloc() {
        User user = new User();
    }

    static class User {
    }

}
```

![image-20220412162906594](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204121629630.png)

​		从jdk 1.7开始已经默认开始逃逸分析.代码中我们循环了100W次User对象,而在堆内存中只有17W个,可以看出逃逸分析生效了.

​		下面把逃逸分析关闭掉.idea启动加入参数,从结果可以看出,没有了逃逸分析,循环100W次,确实是创建了100W个对象在堆内存中.

> -Xmx4G -Xms4G -XX:-DoEscapeAnalysis -XX:+PrintGCDetails -XX:+HeapDumpOnOutOfMemoryError

![image-20220412164115511](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204121641546.png)

![image-20220412163848304](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204121638333.png)

##### 标量替换

- 标量
  一个数据无法再分解为更小的数据来表示了，Java 虚拟机中的基本数据类型 byte、short、int、long、boolean、char、float、double 以及 reference 类型等，都不能再进一步分解了，这些就可以称为标量。
- 聚合量
  一个数据可以继续分解，就称为聚合量。对象就是最典型的聚合量。
  如果把一个 Java 对象拆散，根据程序访问的情况，将其使用到的成员变量恢复到基本数据类型来访问，就叫标量替换。
- 替换过程
  如果逃逸分析可以证明一个对象不会被外部访问，并且这个对象可以拆散的话，那程序真正执行时将可能不创建这个对象，而改为直接创建它的若干个被这个方法使用到的成员变量来替代。将对象拆分后，除了可以让对象的成员变量在栈上分配和读写外（栈上存储的数据，有很大概率会被虚拟机分配至物理机器的高速寄存器中存储），还可以为后续的进一步优化手段创造条件。

> `-XX:+EliminateAllocations` 可以开启标量替换
> `-XX:+PrintEliminateAllocations` 查看标量替换情况（Server VM 非Product 版本支持）

##### 同步锁消除

​		看以下代码示例,在使用StringBuffer时,我们都知道它是线程安全的,也就是它的方法都加了同步锁.加入同步锁肯定会影响执行效率.JIT在开启逃逸分析后,发现StringBuffer对象并没有逃出方法内,所以会使用同步锁消除,也就是以下代码并不会有同步锁加入.

```java
 @Override
    public synchronized StringBuffer append(int i) {
        toStringCache = null;
        super.append(i);
        return this;
    }
```



```java
public class Test {
    public static String getString(String s1, String s2) {
        StringBuffer sb = new StringBuffer();
        sb.append(s1);
        sb.append(s2);
        return sb.toString();
    }

    public static void main(String[] args) {
        long tsStart = System.currentTimeMillis();
        for (int i = 0; i < 10000000; i++) {
            getString("TestLockEliminate ", "Suffix");
        }
        System.out.println("一共耗费：" + (System.currentTimeMillis() - tsStart) + " ms");
    }
}
```

以下执行结果是默认开启了逃逸分析的执行时间

![](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204121645564.png)

 以下执行结果关闭了逃逸分析,执行时间远远超过上边的开启逃逸分析

![](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204121645071.png)



参考文章：

[Java虚拟机-堆](https://blog.csdn.net/meser88/article/details/107917287)

[认识JIT编译器](https://www.cnblogs.com/zumengjie/p/12905330.html)