## JVM虚拟机判断对象“已死”的方法

### 引用计数算法

**实现思路**

​		在对象中添加一个引用计数器，每当有一个地方引用它时，计数器值就加一;当引用失效时，计数器值就减一;任何时刻计数器为零的对象就是不可能再被使用的。

**面临的问题**

​		主流的Java虚拟机里面都没有选用引用计数算法来管理内存，引用计数就很难解决对象之间相互循环引用的问题。

```	java
package org.fenixsoft.jvm.chapter3;

/**
 * testGC()方法执行后，objA和objB会不会被GC呢？
 *
 * @author zzm
 */
public class ReferenceCountingGC {

    public Object instance = null;

    private static final int _1MB = 1024 * 1024;

    /**
     * 这个成员属性的唯一意义就是占点内存，以便在能在GC日志中看清楚是否有回收过
     */
    private byte[] bigSize = new byte[2 * _1MB];

    public static void testGC() {
        ReferenceCountingGC objA = new ReferenceCountingGC();
        ReferenceCountingGC objB = new ReferenceCountingGC();
        objA.instance = objB;
        objB.instance = objA;

        objA = null;
        objB = null;

        // 假设在这行发生GC，objA和objB是否能被回收？
        System.gc();
    }
}
```

### 可达性分析算法

**实现思路**

​		这个算法的基本思路就是通过 一系列称为“GC Roots”的根对象作为起始节点集，从这些节点开始，根据引用关系向下搜索，搜索过程所走过的路径称为“引用链”(Reference Chain)，如果某个对象到GC Roots间没有任何引用链相连， 或者用图论的话来说就是从GC Roots到这个对象不可达时，则证明此对象是不可能再被使用的。

![image-20220413165320489](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204131653529.png)

**可作为GC Roots的对象**

- 在虚拟机栈(栈帧中的本地变量表)中引用的对象，譬如各个线程被调用的方法堆栈中使用到的 参数、局部变量、临时变量等。

- 在方法区中类静态属性引用的对象，譬如Java类的引用类型静态变量。

- 在方法区中常量引用的对象，譬如字符串常量池(String Table)里的引用。

- 在本地方法栈中JNI(即通常所说的Native方法)引用的对象。

- Java虚拟机内部的引用，如基本数据类型对应的Class对象，一些常驻的异常对象(比如NullPointExcepiton、OutOfMemoryError)等，还有系统类加载器。

  所有被同步锁(synchronized关键字)持有的对象。 ·反映Java虚拟机内部情况的JM XBean、JVM TI中注册的回调、本地代码缓存等。

### 加餐

​		经过以上两个算法被判断已经不被使用的对象，还需要经过标记、筛选、再标记的流程才会正真被判为死亡。

**第一次标记：**可达性算法中，对象无法到达`GC Roots`，被标记一次

**筛选：**再被标记的对象中，进行一次筛选，筛选出对象是否有必要执行`finalize`方法，以下是不需要执行的条件：

- 对象没有覆盖finalize()方法
- finalize()方法已经被虚拟机调用过

​		如果对象判定为需要执行`finalize`方法，对象将会被放置在一个名为`F-Queue`的 队列之中，并在稍后由一条由虚拟机自动建立的、低调度优先级的`Finalizer线程`去执行它们的`finalize() `方法

**标记：**收集器将对F-Queue中的对象进行第二次小规模的标记，然后进行回收

> 这个过程中，如果对象要在finaliz e()中成功拯救自己——只要重新与引用链上的任何一个对象建立关联即可，譬如把自己 (this关键字)赋值给某个类变量或者对象的成员变量，那在第二次标记时它将被移出“即将回收”的集合;