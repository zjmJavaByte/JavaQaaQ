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

### **真正”死亡“**

​		经过以上两个算法被判断已经不被使用的对象，还需要经过标记、筛选、再标记的流程才会正真被判为死亡。

**第一次标记：**可达性算法中，对象无法到达`GC Roots`，被标记一次

**筛选：**再被标记的对象中，进行一次筛选，筛选出对象是否有必要执行`finalize`方法，以下是不需要执行的条件：

- 对象没有覆盖finalize()方法
- finalize()方法已经被虚拟机调用过

​		如果对象判定为需要执行`finalize`方法，对象将会被放置在一个名为`F-Queue`的 队列之中，并在稍后由一条由虚拟机自动建立的、低调度优先级的`Finalizer线程`去执行它们的`finalize() `方法

**标记：**收集器将对F-Queue中的对象进行第二次小规模的标记，然后进行回收

> 这个过程中，如果对象要在finaliz e()中成功拯救自己——只要重新与引用链上的任何一个对象建立关联即可，譬如把自己 (this关键字)赋值给某个类变量或者对象的成员变量，那在第二次标记时它将被移出“即将回收”的集合;

**对象的复活-finalize方法的运用**

```java
public class CanReliveObj {
    public static CanReliveObj obj;
    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("CanReliveObj finalize called");
        obj=this;
    }
    @Override
    public String toString(){
        return "I am CanReliveObj";
    }
    public static void main(String[] args) throws InterruptedException{
        obj=new CanReliveObj();
15        obj=null;
        System.gc();
        Thread.sleep(1000);
        if(obj==null){
            System.out.println("obj is null");
        }else{
            System.out.println("obj 可用");
        }
        System.out.println("第2次 gc");
24        obj=null;
        System.gc();
        Thread.sleep(1000);
        if(obj==null){
            System.out.println("obj is null");
        }else{
            System.out.println("obj 可用");
        }
    }
}

```

​		可以看到，在代码第15行将obj设置为`null`后，进行`GC`，结果发现`obj`对象复活了。在 第24行，再次释放对象引用并进行`GC`，对象才真正被回收。这是因为第一次进行`GC`时， 在`finalize()`函数调用之前，虽然系统中的引用已经被清除，但是作为实例方法`finalize()`， 对象的`this`引用依然会被传入方法内部，如果引用外泄，对象就会复活，此时，对象又变 为可触及状态。而`finalize()`函数只会被调用一次，在第2次清除对象时，对象就再无机会 复活，因此就会被回收。

​		注意:`finalize()`函数是一个非常糟糕的模式，不推荐读者使用`finalize()`函数释放资 源。

- 第一，因为`finalize()`函数有可能发生引用外泄，在无意中复活对象; 

- 第二，由于`finalize()`函数是被系统调用的，调用时间是不明确的，因此不是一个好的资源释放方案，推荐在`try-catch-finally`语句中进行资源的释放。

### **并发的可达性分析**（并发标记）

[参考文章](https://mp.weixin.qq.com/s?__biz=Mzg3NjU3NTkwMQ==&mid=2247505094&idx=1&sn=90cfd9f65006ba8c6d96ca4d629506cd&source=41#wechat_redirect) [参考文章](https://mp.weixin.qq.com/s/tWsuQ0HD3RAiKzS-w6giqQ)

​		根节点枚举，在`OopMap`的加持下，所造成的停顿已经是非常短暂且相对固定(不随堆容量而增长)的了。

![](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204151449882.png)

​		可从**GC Roots再继续往下遍历对象 ，这一步骤的停顿时间就必定会与Java堆容量直接成正比例关系了:堆越大，存储的对象越多，对象图结构越复杂，要标记更多对象而产生的停顿时间自然就更长。**

![](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204151449369.png)

​		如何解决这个停顿问题，或者说如何优化呢？就要先搞清楚为什么必须在一个能保障一致性的快照上才能进行对象图的遍历?为了能解释清楚这个问题，我们引入三色标记(Tri-color Marking)[1]作为工具来辅助推导，把遍历对象图过程中遇到的对象，按照“是否访问过”这个条件标记成以下三种颜色:

白色:表示对象尚未被垃圾收集器访问过。显然在可达性分析刚刚开始的阶段，所有的对象都是

- 白色：若在分析结束的阶段，仍然是白色的对象，即代表不可达。

- 黑色：表示对象已经被垃圾收集器访问过，且这个对象的所有引用都已经扫描过。黑色的对象代 表已经扫描过，它是安全存活的，如果有其他对象引用指向了黑色对象，无须重新扫描一遍。黑色对 象不可能直接(不经过灰色对象)指向某个白色对象。

- 灰色：表示对象已经被垃圾收集器访问过，但这个对象上至少存在一个引用还没有被扫描过。

![](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204151451475.png)

​		关于可达性分析的扫描过程，读者不妨发挥一下想象力，把它看作对象图上一股以灰色为波峰的 波纹从黑向白推进的过程，如果用户线程此时是冻结的，只有收集器线程在工作，那不会有任何问 题。但如果用户线程与收集器是并发工作呢?收集器在对象图上标记颜色，同时用户线程在修改引用 关系——即修改对象图的结构，这样可能出现两种后果：

- 一种是把原本消亡的对象错误标记为存活， 这不是好事，但其实是可以容忍的，只不过产生了一点逃过本次收集的**浮动垃圾**而已，下次收集清理 掉就好。

> 浮动垃圾：把原本消亡的对象错误标记为存活

- 另一种是把原本存活的对象错误标记为已消亡，这就是非常致命的后果了，程序肯定会因此 发生错误，下面表3-1演示了这样的致命错误具体是如何产生的。

![image-20220415121822809](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204151218846.png)

​		Wilson于1994年在理论上证明了，当且仅当以下两个条件同时满足时，会产生“对象消失”的问题，即原本应该是黑色的对象被误标为白色:

- 赋值器插入了一条或多条从黑色对象到白色对象的新引用; 
- 赋值器删除了全部从灰色对象到该白色对象的直接或间接引用。

​		因此，我们要解决并发扫描时的对象消失问题，只需破坏这两个条件的任意一个即可。由此分别 产生了两种解决方案:

- **增量更新(Incremental Update)**：要破坏的是第一个条件，当黑色对象插入新的指向白色对象的引用关系时，就将这个新插入的引用记录下来，等并发扫描结束之后，再将这些记录过的引用关系中的黑色对象为根，重新扫描一次。这可以简化理解为，**黑色对象一旦新插入了指向白色对象的引用之后，它就变回灰色对象了。**
-  **原始快照(Snapshot At The Beginning， SAT B)**：要破坏的是第二个条件，当灰色对象要删除指向白色对象的引用关系时，就将这个要删除的引用记录下来，在并发扫描结束之后，再将这些记录过的引用关系中的灰色对象为根，重新扫描一次。这也可以简化理解为，无论引用关系删除与否，都会按照刚刚开始扫描那一刻的对象图快照来进行搜索。

​		**以上无论是对引用关系记录的插入还是删除，虚拟机的记录操作都是通过写屏障实现的**

**写屏障**

- 这里的写屏障和我们常说的为了解决并发乱序执行问题的"内存屏障"不是一码事，需要区分开来。

- 写屏障可以看作虚拟机层面对"引用类型字段赋值"这个动作的AOP切面，在引用对象赋值时会产生一个环形通知，供程序执行额外的动作，也就是说赋值的前后都在写屏障的覆盖范畴内。在赋值前的部分的写屏障叫做写前屏障(Pre-Write Barrier)，在赋值后的则叫作写后屏障(Post-Write Barrier)。

所以，经过简单的推导我们可以知道：

​		**增量更新用的是写后屏障(Post-Write Barrier)，记录了所有新增的引用关系。**

​		**原始快照用的是写前屏障(Pre-Write Barrier)，将所有即将被删除的引用关系的旧引用记录下来。**