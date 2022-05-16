[TOC]

## 对象头和锁

​		要理解轻量级锁，以及后面会讲到的偏向锁的原理和运作过程，必须要对`HotSpot`虚拟机对象的内 存布局(尤其是对象头部分)有所了解。HotSpot虚拟机的对象头`(Object Header)`分为两部分，第一 部分用于存储对象自身的运行时数据，如哈希码`(HashCode)`、`GC`分代年龄`(Generational GC Age) `等。这部分数据的长度在32位和64位的Java虚拟机中分别会占用32个或64个比特，官方称它为`Mark Word`。这部分是实现轻量级锁和偏向锁的关键。另外一部分用于存储指向方法区对象类型数据的指针，如果是数组对象，还会有一个额外的部分用于存储数组长度。（[HotSpot虚拟机对象](https://github.com/zjmJavaByte/JavaQaaQ/blob/master/docs/jdk/HotSpot%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%AF%B9%E8%B1%A1.md)）

​		由于对象头信息是与对象自身定义的数据无关的额外存储成本，考虑到Java虚拟机的空间使用效 率，`Mark Word`被设计成一个非固定的动态数据结构，以便在极小的空间内存储尽量多的信息。它会 根据对象的状态复用自己的存储空间。例如在32位的`HotSpot`虚拟机中，对象未被锁定的状态下， `Mark Word`的32个比特空间里的25个比特将用于存储**对象哈希码**，4个比特用于存储**对象分代年龄**，2 个比特用于存储**锁标志位**，还有1个比特固定为0(这表示未进入偏向模式)。对象除了未被锁定的正常状态外，还有**轻量级锁定、重量级锁定、GC标记、可偏向**等几种不同状态，这些状态下对象头的存 储内容如表所示

![image-20220506154803018](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205071441257.png)

## 偏向锁

​		**如果说轻量级锁是在无竞争的情况下使用CAS操作去消除同步使用的互斥量，那偏向锁就是在无竞争的情况下把整个同步都消除掉，连CAS操作都不去做了。**

​		这个锁会偏向于第一个获得它的线 程，如果在接下来的执行过程中，该锁一直没有被其他的线程获取，则持有偏向锁的线程将永远不需 要再进行同步。

**偏向锁的加锁工作过程**

​		假设当前虚拟机启用了偏向锁(启用参数`-XX:+UseBiasedLocking`，这是自JDK 6 起`HotSpot`虚拟机的默认值)，那么当锁对象第一次被线程获取的时候，虚拟机将会把对象头中的标志 位设置为`01`、把偏向模式设置为`1`，表示进入偏向模式。同时使用CAS操作把获取到这个锁的线程 的ID记录在对象的`Mark Word`之中。如果`CAS`操作成功，持有偏向锁的线程以后每次进入这个锁相关 的同步块时，虚拟机都可以不再进行任何同步操作(例如加锁、解锁及对`Mark Word`的更新操作等)。

**偏向锁的撤销**

​		一旦出现另外一个线程去尝试获取这个锁的情况，偏向模式就马上宣告结束。根据锁对象目前是 否处于被锁定的状态决定是否撤销偏向(偏向模式设置为`0`)，撤销后标志位恢复到未锁定(标志位 为` 01`)或轻量级锁定(标志位为` 00`)的状态，后续的同步操作就按照下面介绍的轻量级锁那样去执行。偏向锁、轻量级锁的状态转化及对象`Mark Word`的关系如图

![image-20220506160620342](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205061606381.png)

​		细心的读者看到这里可能会发现一个问题:当对象进入偏向状态的时候，`Mark Word`大部分的空间(23个比特)都用于存储持有锁的线程`ID`了，这部分空间占用了原有存储对象哈希码的位置，那原来对象的哈希码怎么办呢?

​		在Java语言里面一个对象如果计算过哈希码，就应该一直保持该值不变(强烈推荐但不强制，因为用户可以重载`hashCode()`方法按自己的意愿返回哈希码)，否则很多依赖对象哈希码的`API`都可能存在出错风险。而作为绝大多数对象哈希码来源的`Object::hashCode()`方法，返回的是对象的一致性哈希码(`Identity Hash Code`)，这个值是能强制保证不变的，它通过在对象头中存储计算结果来保证第一 次计算之后，再次调用该方法取到的哈希码值永远不会再发生改变。因此，当一个对象已经计算过一 致性哈希码后，它就再也无法进入偏向锁状态了;而当一个对象当前正处于偏向锁状态，又收到需要计算其一致性哈希码请求时，它的偏向状态会被立即撤销，并且锁会膨胀为重量级锁。在重量级锁 的实现中，对象头指向了重量级锁的位置，代表重量级锁的`ObjectMonitor`类里有字段可以记录非加锁状态(标志位为`01`)下的`Mark Word`，其中自然可以存储原来的哈希码。

​		下面这段代码可以展示使用偏向锁之后的性能提升，在笔者的测试中， 使用偏向锁简化了锁的处理流程，可以获得大约20%的性能提升。

```java
/**
 * @author Geym
 * -XX:+UseBiasedLocking 
	 -XX:BiasedLockingStartupDelay=0
	 -client
	 -Xmx512m
	 -Xms512m
 */
public class Biased {
    public static List<Integer> numberList =new Vector<Integer>();
    public static void main(String[] args) throws InterruptedException {
        long begin=System.currentTimeMillis();
        int count=0;
        int startnum=0;
        while(count<10000000){
            numberList.add(startnum);
            startnum+=2;
            count++;
        }
        long end=System.currentTimeMillis();
        System.out.println(end-begin);
    }
}
```

> 228

​		这说明程序用228毫秒完成所有工作。参数中的`-XX:BiasedLockingStartupDelay`表示虚拟机在启动后立即启用偏向锁。如不设置该参数，虚拟机默认会在启动后4秒后，才启用偏向锁，考虑到程序运行时间较短，故做此设置，尽早启用偏向锁。

若禁用偏向锁，则只需使用如下参数启动程序:

> -XX:-UseBiasedLocking
> -client
> -Xmx512m
> -Xms512m

> 298

​		可以看到，偏向锁在竞争少的情况下，对系统性能有一定的帮助。

​		偏向锁在锁竞争激烈的场合没有太强的优化效果，因为大量的竞争会导致持有锁的线 程不停地切换，锁也很难一直保持在偏向模式，此时使用锁偏向不仅得不到性能优化，反 而有可能降低系统性能。因此，在竞争激烈的场合，可以尝试使用`-XX:-UseBiasedLocking` 参数禁用偏向锁。

## 轻量级锁

​		轻量级锁并不是 用来代替重量级锁的，它设计的初衷是在**没有多线程竞争的前提下**，减少传统的重量级锁使用操作系统互斥量产生的性能消耗。

**轻量级锁的加锁工作过程**:

​		在代码即将进入同步块的时候，如果此同步对象没有被锁定(锁标志位为` 01`状态)，虚拟机首先将在当前线程的栈帧中建立一个名为锁记录(`Lock Record`)的空间，用于存储锁对象目前的`Mark Word`的拷贝(官方为 这份拷贝加了一个`Displaced`前缀，即`Displaced Mark Word`)，这时候线程堆栈与对象头的状态如图

![image-20220506154850517](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205061548544.png)

​		然后，虚拟机将使用`CAS`操作尝试把对象的`Mark Word`更新为指向`Lock Record`的指针。如果这个更新动作成功了，即代表该线程拥有了这个对象的锁，并且对象`Mark Word`的锁标志位(`Mark Word`的 最后两个比特)将转变为`00`，表示此对象处于轻量级锁定状态。这时候线程堆栈与对象头的状态如图

![image-20220506154930561](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205061549589.png)

​		如果这个更新操作失败了，那就意味着至少存在一条线程与当前线程竞争获取该对象的锁。虚拟 机首先会检查对象的`Mark Word`是否指向当前线程的栈帧，如果是，说明当前线程已经拥有了这个对 象的锁，那直接进入同步块继续执行就可以了，否则就说明这个锁对象已经被其他线程抢占了。如果 出现两条以上的线程争用同一个锁的情况，那轻量级锁就不再有效，必须要膨胀为重量级锁，锁标志 的状态值变为`10`，此时`Mark Word`中存储的就是指向重量级锁(互斥量)的指针，后面等待锁的线 程也必须进入阻塞状态。

**轻量级锁解锁**

​		上面描述的是轻量级锁的加锁过程，它的解锁过程也同样是通过CAS操作来进行的，如果对象的 `Mark Word`仍然指向线程的锁记录，那就用CAS操作把对象当前的`Mark Word`和线程中复制的`Displaced Mark Word`替换回来。假如能够成功替换，那整个同步过程就顺利完成了;如果替换失败，则说明有 其他线程尝试过获取该锁，就要在释放锁的同时，唤醒被挂起的线程。

**争夺锁导致的锁膨胀流程图**

![image-20220508143500870](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205081435012.png)

​		**轻量级锁能提升程序同步性能的依据是“对于绝大部分的锁，在整个同步周期内都是不存在竞争 的”这一经验法则。**如果没有竞争，轻量级锁便通过`CAS`操作成功避免了使用互斥量的开销;但如果确 实存在锁竞争，除了互斥量的本身开销外，还额外发生了`CAS`操作的开销。因此在有竞争的情况下， 轻量级锁反而会比传统的重量级锁更慢。

## 锁粗化

​		**如果一系列的连续操作都对同一个对象反复加锁和解锁，甚至加锁操作是出现在循环体之中的，那即使没有线程竞争，频繁地进行互斥同步操作也会导致不必要的性能损耗。**

​		如果虚拟机探测到有这样一串零碎的操作 都对同一个对象加锁，将会把加锁同步的范围扩展(粗化)到整个操作序列的外部，以代码清单为例，就是扩展到第一个`append()`操作之前直至最后一个`append()`操作之后，这样只需要加锁一次就可 以了。

```java
public String concatString(String s1, String s2, String s3) { 				StringBuffer sb = new StringBuffer();
		sb.append(s1);
		sb.append(s2);
		sb.append(s3);
		return sb.toString(); 
}
```

## 自旋锁与自适应自旋

​		互斥同步对性能最大的影响是阻塞的实现，**挂起线程和恢复线程的操作都需要转入内核态中完成**，这些操作给Java虚拟机的并发性能带来了很大的压力。（要理解这计划可以先看[Java线程的实现](https://zhuanlan.zhihu.com/p/358483053)）

​		自旋锁在JDK 1.4.2中就已经引入，只不过默认是关闭的，可以使用`-XX:+UseSpinning`参数来开 启，在JDK 6中就已经改为默认开启了。**自旋等待不能代替阻塞，且先不说对处理器数量的要求，自旋等待本身虽然避免了线程切换的开销，但它是要占用处理器时间的，所以如果锁被占用的时间很短，自旋等待的效果就会非常好，反之如果锁被占用的时间很长，那么自旋的线程只会白白消耗处理 器资源，而不会做任何有价值的工作，这就会带来性能的浪费**。因此自旋等待的时间必须有一定的限度，如果自旋超过了限定的次数仍然没有成功获得锁，就应当使用传统的方式去挂起线程。自旋次数 的默认值是十次，用户也可以使用参数`-XX:PreBlockSpin`来自行更改。

​		**在 JDK 6中对自旋锁的优化，引入了自适应的自旋。自适应意味着自旋的时间不再是固定的了，而是由 前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定的。**如果在同一个锁对象上，自旋等待刚 刚成功获得过锁，并且持有锁的线程正在运行中，那么虚拟机就会认为这次自旋也很有可能再次成 功，进而允许自旋等待持续相对更长的时间，比如持续100次忙循环。另一方面，如果对于某个锁，自 旋很少成功获得过锁，那在以后要获取这个锁时将有可能直接省略掉自旋过程，以避免浪费处理器资 源。

## 锁消除

​		**锁消除是指虚拟机即时编译器在运行时检测到某段需要同步的代码根本不可能存在共享数据竞争而实施的一种对锁进行消除的优化策略。**锁消除的主要判定依据来源于逃逸分析(开启逃逸分析`-XX:+DoEscapeAnalysis`)的数据支持，如果判断到一段代码中，在堆上的所有数据都不会逃逸出去被其他线程访问到，那就可 以把它们当作栈上数据对待，认为它们是线程私有的，同步加锁自然就无须再进行。

​		也许读者会有疑问，变量是否逃逸，对于虚拟机来说是需要使用复杂的过程间分析才能确定的， 但是程序员自己应该是很清楚的，怎么会在明知道不存在数据争用的情况下还要求同步呢?这个问题 的答案是:有许多同步措施并不是程序员自己加入的，同步的代码在Java程序中出现的频繁程度也许 超过了大部分读者的想象。我们来看看如代码清单所示的例子，这段非常简单的代码仅仅是输出 三个字符串相加的结果，无论是源代码字面上，还是程序语义上都没有进行同步。

```java
public String concatString(String s1, String s2, String s3) { 				return s1 + s2 + s3;
}
```

​		我们也知道，由于`String`是一个不可变的类，对字符串的连接操作总是通过生成新的`String`对象来 进行的，因此`Javac`编译器会对`String`连接做自动优化。在JDK 5之前，字符串加法会转化为`StringBuffer` 对象的连续`append()`操作，在JDK 5及以后的版本中，会转化为`StringBuilder`对象的连续`append()`操作。

```java
public String concatString(String s1, String s2, String s3) { 				StringBuffer sb = new StringBuffer();
		sb.append(s1);
		sb.append(s2);
		sb.append(s3);
		return sb.toString(); 
}
```

​		现在大家还认为这段代码没有涉及同步吗?每个`StringBuffer.append()`方法中都有一个同步块，锁 就是sb对象。虚拟机观察变量sb，经过逃逸分析后会发现它的动态作用域被限制在`concatString()`方法内 部。也就是`sb`的所有引用都永远不会逃逸到`concatString()`方法之外，其他线程无法访问到它，所以这里 虽然有锁，但是可以被安全地消除掉(开启所消除`-XX:+EliminateLocks`,锁消除必须工作在`-server`模式下)。在解释执行时这里仍然会加锁，但在经过服务端编译器的即时 编译之后，这段代码就会忽略所有的同步措施而直接执行。

## 锁的优缺点对比

![image-20220508143858789](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205081438834.png)

## synchronized底层实现

​		关键字`synchronized`可以修饰方法或者以同步块的形式来进行使用，它主要确保多个线程 在同一个时刻，只能有一个线程处于方法或者同步块中，它保证了线程对变量访问的可见性和排他性。

```java
public class Synchronized {
    public static void main(String[] args) {
        // Synchronized Class对象进行加锁
        synchronized (Synchronized.class) {

        }
        // 静态同步方法，对Synchronized Class对象进行加锁
        m();
    }

    public static synchronized void m() {
    }
}
```

​		通过`javapc–v Synchronized.class`查看编译后的结果：

>   public static void main(java.lang.String[]);
>     descriptor: ([Ljava/lang/String;)V
>     flags: ACC_PUBLIC, ACC_STATIC
>     Code:
>       stack=2, locals=3, args_size=1
>          0: ldc           #2                  // class com/zjmByte/ArtConcurrentBook/chapterFour/Synchronized
>          2: dup
>          3: astore_1
>          4: **monitorenter**
>          5: aload_1
>          6: **monitorexit**
>          7: goto          15
>         10: astore_2
>         11: aload_1
>         12: monitorexit
>         13: aload_2
>         14: athrow
>         15: invokestatic  #3                  // Method m:()V
>         18: return
>       Exception table:
>          from    to  target type
>              5     7    10   any
>             10    13    10   any
>       LineNumberTable:
>         line 7: 0
>         line 9: 5
>         line 11: 15
>         line 12: 18
>       StackMapTable: number_of_entries = 2
>         frame_type = 255 /* full_frame */
>           offset_delta = 10
>           locals = [ class "[Ljava/lang/String;", class java/lang/Object ]
>           stack = [ class java/lang/Throwable ]
>         frame_type = 250 /* chop */
>           offset_delta = 4
>
>   public static synchronized void m();
>     descriptor: ()V
>     flags: ACC_PUBLIC, ACC_STATIC, **ACC_SYNCHRONIZED**
>     Code:
>       stack=0, locals=0, args_size=0
>          0: return
>       LineNumberTable:
>         line 15: 0
> }

​		上面`class`信息中，对于同步块的实现使用了`monitorenter和monitorexit`指令，而同步方法则是依靠方法修饰符上的`ACC_SYNCHRONIZED`来完成的。无论采用哪种方式，其本质是对一 个对象的监视器(`monitor`)进行获取，而这个获取过程是排他的，也就是同一时刻只能有一个线程获取到由`synchronized`所保护对象的监视器。

​		任意一个对象都拥有自己的监视器，当这个对象由同步块或者这个对象的同步方法调用 时，执行方法的线程必须先获取到该对象的监视器才能进入同步块或者同步方法，而没有获 取到监视器(执行该方法)的线程将会被阻塞在同步块和同步方法的入口处，进入`BLOCKED` 状态。对象、对象的监视器、同步队列和执行线程之间的关系。

![image-20220510112408286](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205101124305.png)

​		任意线程对`Object(Object由synchronized`保护)的访问，首先要获得 `Object`的监视器。如果获取失败，线程进入同步队列，线程状态变为`BLOCKED`。当访问`Object `的前驱(获得了锁的线程)释放了锁，则该释放操作唤醒阻塞在同步队列中的线程，使其重新 尝试对监视器的获取。
