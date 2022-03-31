# JDK8



# JDK9

# JDK10



# JDK11

###### 1、局部变量类型推断（Local Var）

在Lambda表达式中，可以使用var关键字来标识变量，变量类型由编译器自行推断。

```java
// LocalVar.java
import java.util.Arrays;
public class LocalVar {
    public static void main(String[] args) {
        Arrays.asList("Java", "Python", "Ruby")
            .forEach((var s) -> {
                System.out.println("Hello, " + s);
            });
    }
}
```

###### 2、增加一些实用的API

增加了许多字符串自带方法

```java
@Test
public void contextLoads() {
    String str = "woshidage";
    boolean isblank = str.isBlank();  //判断字符串是空白
    boolean isempty = str.isEmpty();  //判断字符串是否为空
    String  result1 = str.strip();    //首位空白 strip()可以去掉Unicode空格，例如，中文空格：
    String  result2 = str.stripTrailing();  //去除尾部空白
    String  result3 = str.stripLeading();  //去除首部空白
    String  copyStr = str.repeat(2);  //复制几遍字符串
    long  lineCount = str.lines().count();  //行数统计

    System.out.println(isblank);
    System.out.println(isempty);
    System.out.println(result1);
    System.out.println(result2);
    System.out.println(result3);
    System.out.println(copyStr);
    System.out.println(lineCount);
}
```

###### 3、新的Epsilon垃圾收集器

 A NoOp Garbage Collector JDK上对这个特性的描述是: 开发一个处理内存分配但不实现任何实际内存回收机制的GC, 一旦可用堆内存用完, JVM就会退出.

如果有System.gc()调用, 实际上什么也不会发生(这种场景下和-XX:+DisableExplicitGC效果一样), 因为没有内存回收, 这个实现可能会警告用户尝试强制GC是徒劳.

   用法 : -XX:+UnlockExperimentalVMOptions -XX:+UseEpsilonGC

   选项-XX:+UseEpsilonGC, 程序很快就因为堆空间不足而退出

   原因 :提供完全被动的GC实现, 具有有限的分配限制和尽可能低的延迟开销,但代价是内存占用和内存吞吐量, java实现可广泛选择高度可配置的GC实现. 各种可用的收集器最终满足不同的需求, 即使它们的可配置性使它们的功能相交. 有时更容易维护单独的实现, 而不是在现有GC实现上堆积另一个配置选项.

主要用途如下 :

```java
//下面四点完全诠释了这句话：内存分配但不实现任何实际内存回收机制的GC, 一旦可用堆内存用完, JVM就会退出
1、性能测试(它可以帮助过滤掉GC引起的性能假象)

2、内存压力测试(例如,知道测试用例 应该分配不超过1GB的内存, 我们可以使用-Xmx1g –XX:+UseEpsilonGC, 
   如果程序有问题, 则程序会崩溃)

3、非常短的JOB任务(对象这种任务, 接受GC清理堆那都是浪费空间)

4、VM接口测试

5、Last-drop 延迟&吞吐改进
```

###### 4、ZGC

ZGC, 这应该是JDK11最为瞩目的特性, 没有之一. 但是后面带了Experimental, 说明这还不建议用到生产环境。

```Java
1、GC暂停时间不会超过10ms，既能处理几百兆的小堆, 也能处理几个T的大堆(OMG)，和G1相比, 应用吞吐能力不会下降超过15%，为未来的GC功能和利用colord指针以及Load barriers优化奠定基础，初始只支持64位系统

2、ZGC的设计目标是：支持TB级内存容量，暂停时间低（<10ms），对整个程序吞吐量的影响小于15%。 将来还可以扩展实现机制，以支持不少令人兴奋的功能，例如多层堆（即热对象置于DRAM和冷对象置于NVMe闪存），或压缩堆。

3、GC是java主要优势之一，当GC停顿太长, 就会开始影响应用的响应时间.消除或者减少GC停顿时长, java将对更广泛的应用场景是一个更有吸引力的平台. 此外, 现代系统中可用内存不断增长,用户和程序员希望JVM能够以高效的方式充分利用这些内存, 并且无需长时间的GC暂停时间
```

> 用法 : -XX:+UnlockExperimentalVMOptions –XX:+UseZGC, 因为ZGC还处于实验阶段, 所以需要通过JVM参数来解锁这个特性

# JDK12



# JDK13

# JDK14

# JDK15

# JDK16