# **String**在虚拟机中的实现

​		`String`字符串一直是各种编程语言的核心。字符串应用很广泛，每一种计算机语言都 必须对其做特殊的优化和实现。在`Java`中，`String`虽然不是基本数据类型，但是也享有了 和基本数据类型一样的待遇。

## **String**对象的特点

​		在Java语言中，Java的设计者对String对象进行了大量的优化，主要表现在以下3个方面，同时这也是String对象的3个基本特点: · 

- 不变性。

​		不变性是指`String`对象一旦生成，则不能再对它进行改变。`String`的这个特性可以泛化 成不变`(immutable)`模式，即一个对象的状态在对象被创建之后就不再发生变化。不变 模式的主要作用在于，当一个对象需要被多线程共享并且访问频繁时，可以省略同步和锁 等待的时间，从而大幅提高系统性能。

  		注意:不变性可以提高多线程访问的性能。因为对象不可变，对于所有线程都是只读的，多线程访问时，即使不加同步也不会导致数据不一致，减小了系统开销。

​		由于不变性，一些看起来像修改的操作，实际上都是依靠产生新的字符串实现的。比 如`String.substring()、String.concat()`方法，它们都没有修改原始字符串，而是产生了一个新 的字符串，这一点是非常值得注意的。如果需要一个可以修改的字符串，那么需要使用 `StringBuffer或者StringBuilder`对象。

- 针对常量池的优化。

​		针对常量池的优化指当两个`String`对象拥有相同的值时，它们只引用常量池中的同一个副本。当同一个字符串反复出现时，这个技术可以大幅度节省内存空间。

```java
public static void main(String[] args) {
        String str1 = new String("abc");
        String str2 = new String("abc");
        System.out.println(str1 == str2);//false
        System.out.println(str1 == str2.intern());//false
        System.out.println(str1.intern() == str2.intern());//true
}
```

​		以上代码`str1`和`str2`都开辟了一块堆空间存放`String`实例，如下图所示。虽然`str1`和`str2 `内容相同，但是在堆中的引用是不同的。`String.intern()`返回字符串在常量池中的引用，显 然它和`str1`也是不同的，但是根据最后一行代码可以知道，`String.intern()`始终和常量字符 串相等，读者可以思考一下，`str1.intern()与str2.intern()`是否相等呢?

![image-20220507112845740](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205071128153.png)

- 类的final定义。

​		除以上两点外，`final`类型定义也是`String`对象的重要特点。作为`final`类的String对象在系统中不可能有任何子类，这是对系统安全性的保护。同时，在`JDK 1.5`版本之前的环境 中，使用`final`定义有助于帮助虚拟机寻找机会，[内联](https://github.com/zjmJavaByte/JavaQaaQ/blob/master/docs/jdk/%E9%80%83%E9%80%B8%E5%88%86%E6%9E%90%E3%80%81%E6%A0%87%E9%87%8F%E6%9B%BF%E6%8D%A2%E3%80%81%E6%A0%88%E4%B8%8A%E5%88%86%E9%85%8D.md#%E6%96%B9%E6%B3%95%E5%86%85%E8%81%94)所有的`final`方法，从而提高系统效 率。但这种优化方法在`JDK 1.5`以后，效果并不明显。

## 有关**String**常量池的位置

​		在虚拟机中，有一块称为常量池的区域专门用于存放字符串常量。在JDK 1.6之前，这块区域属于永久区的一部分，但是在JDK 1.7以后，它就被移到了堆中进行管理。

```java
/**
     * -Xmx10M
     * @param args
     */
    public static void main(String[] args) {
        ArrayList<String> strings = new ArrayList<>();
        int i = 0;
        while (true){
            strings.add(String.valueOf(++i).intern());
        }
    }
```

​		上述代码使用`String.intern()`获得常量池中的字符串引用，如果常量池中没有该常量字 符串，该方法会将字符串加入常量池。然后，将该引用放入list进行持有，确保不被回 收。使用如下参数运行这段程序:

> -Xmx5m -XX:MaxPermSize=5m

在JDK 1.6中抛出错误如下:

![image-20220507113351716](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205071133756.png)

在JDK 1.7~JDK 1.10中抛出错误如下:

![image-20220507113328567](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205071133602.png)

​		溢出的区域已经不同，JDK 1.6中发生在永久区，而JDK 1.7~JDK1.10中则发生在堆 中。这间接表明了常量池的位置变化。

​		另外一点值得注意的是，虽然String.intern()的返回值永远等于字符串常量。但这 并不代表在系统的每时每刻，相同的字符串的intern()返回值都会是一样的(虽然在 95%以上的情况下，都是相同的)。因为存在这么一种可能:在一次intern()调用之 后，该字符串在某一个时刻被回收，之后，再进行一次intern()调用，那么字面量相同 的字符串重新被加入常量池，但是引用位置已经不同。

```java
public class ConstantPool {
    public static void main(String[] args) {
        if(args.length==0)return;
        System.out.println(System.identityHashCode((args[0]+Integer.toString(0))));
        System.out.println(System.identityHashCode((args[0]+Integer.toString(0)).intern()));
        System.gc();
        System.out.println(System.identityHashCode((args[0]+Integer.toString(0)).intern()));
     }
}
```

​		上述代码接收一个参数，用于构造字符串，构造的字符串都是在原有字符串后加上字 符串“0”。一共输出3次字符串的Hash值:第一次为字符串本身，第二次为常量池引用，第 三次为进行了常量池回收后的相同字符串的常量池引用。程序的一种可能输出如下:

> 3916375
>
> 22279806
>
> 3154093

​		可以看到，3次Hash值都是不同的。但是如果不进行程序中的显式GC操作，那么后两 次Hash值理应是相同的。