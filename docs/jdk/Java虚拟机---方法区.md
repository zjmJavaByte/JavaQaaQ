### Java虚拟机---方法区

#### 栈、堆、方法区 的相关关系

从线程共享角度来看

![](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204121741092.png)

交互关系

![](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204121741018.png)

> 上图：Person.class 对象是存储在方法区中，person局部变量是在Java栈中，new 的 Person 对象是存储在 Java 堆中。

![](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204121742084.png)

> 上图：Java栈中有一个 reference 的引用，引用到Java堆的具体对象中，对象中会存有具体方法区中的类对象的类型。

#### 方法区的理解

​		方法区，与Java堆一样，是一块独立的空间，也是各个线程共享的内存区域，它用于存储已被虚拟机加载的类型信息、常量、静态变量、即时编译器编译后的代码缓存等数据。

![](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204121742650.png)


注：随着JDK版本的更新，JDK8 完成了永久代的废弃，**通过本地缓存实现的元空间**，将上述信息全部保存在了元空间中。

**类型信息：对每个加载的类型（类class、接口interface、枚举enum、注解annotation），jvm必须在方法区中存储一下信息：**

> 1.这个类型的完整有效名称（全名=包名.类名）
> 2.这个类型的直接父类的完整有效名（对于interface或者java.lang.Object，都没有父类）
> 3.这个类型的修饰符（public、abstract、final的某个子集）
> 4.这个类型直接接口的一个有序列表

**域信息：（成员变量，属性）**

> jvm必须在方法区中保存类型的所有域的相关信息以及域的生命顺序。
> 域的相关信息包括：域名城、域类型、域修饰符（public、private、protected、static、final、volatile、transient的某个子集）

**方法的信息（Method）**

> 方法名称
> 方法的返回类型或者void
> 方法参数的数量和类型（按照顺序）
> 方法的修饰符（public、private、protected、static、final、synchronized、transient的某个子集）
> 方法的字节码、操作数栈、局部变量表以及大小（abstract和native除外）
> 异常表（abstract和native除外）：每个异常处理的开始位置、结束位置、代码处理在程序计数器中的偏移地址、被捕获的异常类的常量池索引。

可以通过 javap 查看一些字节码都有哪些信息。

​		全局常量：static final 修饰：被声明为final的类变量的处理方式则不同，每个全局常量在编译的时候就被分配了。
如下面的例子：

```java
public class Test {
    public static int a = 1;
    public static final int b = 2;
public Test() { /* compiled code */ }
}
```
​		通过javap 后有下面几行：final 修饰的 b 变量在编译的时候就已经赋值了。
非final 修饰的变量 a 在类的 prepare阶段 初始化的 0 ，然后在 initialization 的时候赋值的 1

```java
public static int a;
    descriptor: I
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC

  public static final int b;
    descriptor: I
    flags: (0x0019) ACC_PUBLIC, ACC_STATIC, ACC_FINAL
    ConstantValue: int 2
```

#### 常量池

​		一个有效的字节码文件中除了包含类的版本信息、字段、方法以及接口等描述信息外，还包含一项信息那就是常量池表，包括各种字面量和对类型，域和方法符号引用。

​		字节码常量池表在类加载以后放到运行时常量池中，这就是运行时常量池。
例：还是上面那段代码通过 JclassLib查看反编译以后如下图：

![](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204121744250.png)

常量池部分截图如下：

![](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204121745114.png)

​		上面字节码中 类似于 #30 对应的常量池的[30]，就这样一一对应的执行顺序。
比如上面 [01]调用了 #2 和 #3 。

​		常量池内存储的数据类型包括：数量值、字符串值、类引用、字段引用、方法引用

​		常量池受到方法区大小的限制，当无法申请内存的时候，会出现OOM异常。
总结：常量池可以看作一张表，虚拟机指令根据这张常量表找到要执行的类名、方法名、参数类型、字面量类型等。

#### 运行时常量池

- 运行时常量池是方法区中的一部分。常量池表是Class文件的一部分，用于存放编译期生成的各种字面量和符号引用，这部分内容将在类加载后存放到方法区的运行时常量池中。

- 运行时常量池，在加载类和接口到虚拟机后，就会创建对应的运行时常量池。
  JVM为每个已加载的类型（类或接口）都维护一个常量池。池中的数据就像数组一样，通过索引访问的。
  
- 运行时常量池中包含多种不同的常量，包括编译期就已经明确的数值字面量，也包括运行期解析后才能够获得的方法或者字段引用。此时不再是常量池中的符号地址了，这里换为真实地址。

- 运行时常量池相对于Class文件的常量池的重要特征是：具有动态性。（例：String.intern()）

- 运行时常量池类似于传统的编程语言中的符号表（symbol table），但是它所包含的数据比符号表更加丰富一些。

- 当创建类或者接口的运行时常量池时，如果构造运行时常量池所需的内存空间超过了方法区所能承受的最大值。则 OOM。

#### 方法区的垃圾回收

  		HotSpot虚拟机实现了垃圾回收机制，主要回收内容有两部分：常量池中废弃的常量和不再使用的类型。

常量的回收：

![](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204121746438.png)

类的回收：

![](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204121746582.png)

————————————————
版权声明：本文为CSDN博主「ldxlz224」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/ldxlz224/article/details/116804015