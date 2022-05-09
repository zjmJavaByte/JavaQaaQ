[TOC]

# final域的内存语义

## final域的重排序规则

- 在构造函数内对一个`final`域的写入，与随后把这个被构造对象的引用赋值给一个引用 变量，这两个操作之间不能重排序。
- 初次读一个包含`final`域的对象的引用，与随后初次读这个`final`域，这两个操作之间不能 重排序。

### final修饰的基本类型

```java
public class FinalExample {
    int                 i;  //普通变量
    final int           j;  //final变量
    static FinalExample obj;

    public FinalExample() { //构造函数
        i = 1; //写普通域
        j = 2; // 写final域
    }

    public static void writer() { //写线程A执行
        obj = new FinalExample();
    }

    public static void reader() { //读线程B执行
        FinalExample object = obj; //读对象引用
        int a = object.i; //读普通域
        int b = object.j; //读final域
    }
}
```

#### 写final域的重排序规则

​		写`final`域的重排序规则禁止把final域的写重排序到构造函数之外。

- `JMM`禁止编译器把final域的写重排序到构造函数之外。
- 编译器会在`final`域的写之后，构造函数`return`之前，插入一个`StoreStore`屏障。这个屏障 禁止处理器把`final`域的写重排序到构造函数之外。

**分析writer()方法**

- 构造一个FinalExample类型的对象。

- 把这个对象的引用赋值给引用变量`obj`。

​		如下图所示：写普通域的操作被编译器重排序到了构造函数之外，读线程B错误地读取了 普通变量i初始化之前的值。而写final域的操作，被写final域的重排序规则“限定”在了构造函数 之内，读线程B正确地读取了final变量初始化之后的值。

![image-20220509135505516](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205091355236.png)

**写`final`域的重排序规则**

​		在对象引用为任意线程可见之前，对象的`final`域已经被 正确初始化过了，而普通域不具有这个保障。以上图为例，在读线程`B`“看到”对象引用`obj`时， 很可能`obj`对象还没有构造完成(对普通域i的写操作被重排序到构造函数外，此时初始值`1`还 没有写入普通域i)。

#### 读final域的重排序规则

​		读`final`域的重排序规则是：在一个线程中，初次读对象引用与初次读该对象包含的`final `域，`JMM`禁止处理器重排序这两个操作(注意，这个规则仅仅针对处理器)。编译器会在读`final `域操作的前面插入一个`LoadLoad`屏障。

- 初次读对象引用与初次读该对象包含的`final`域，这两个操作之间存在间接依赖关系。

**分析reader()方法**

- 初次读引用变量`obj`。 

- 初次读引用变量`obj`指向对象的普通域j。 
- 初次读引用变量`obj`指向对象的`final`域`i`。

​		如下图所示，读对象的普通域的操作被处理器重排序到读对象引用之前。读普通域时，该 域还没有被写线程`A`写入，这是一个错误的读取操作。而读`final`域的重排序规则会把读对象` final`域的操作“限定”在读对象引用之后，此时该`final`域已经被A线程初始化过了，这是一个正 确的读取操作。

![image-20220509140201618](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205091402657.png)

**读final域的重排序规则**

​		在读一个对象的`final`域之前，一定会先读包含这个`final` 域的对象的引用。

### final修饰的引用类型

本例`final`域为一个引用类型，它引用一个`int`型的数组对象。

```java
public class FinalReferenceExample {
    final int[] intArray; //final是引用类型
    static FinalReferenceExample obj;

    public FinalReferenceExample() { //构造函数
        intArray = new int[1]; //1
        intArray[0] = 1; //2
    }

    public static void writerOne() { //写线程A执行
        obj = new FinalReferenceExample(); //3
    }

    public static void writerTwo() { //写线程B执行
        obj.intArray[0] = 2; //4
    }

    public static void reader() { //读线程C执行
        if (obj != null) { //5
            int temp1 = obj.intArray[0]; //6
        }
    }
}
```

#### 写final域的重排序规则

​		在构造函数内对一个`final`引用的对象的成员域的写入，与随后在构造函数外把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。

**分析writerOne()方法**

​		`1`是对`final`域的写入，`2`是对这个`final`域引用的对象的成员域的写入，`3`是把被构造的对象的引用赋值给某个引用变量。这里除了前面提到的`1`不能和`3`重排序外，`2`和`3`也不能重排序。

​		对上面的示例程序，假设首先线程`A`执行`writerOne()`方法，执行完后线程`B`执行` writerTwo()`方法，执行完后线程C执行reader()方法。如下图是一种可能的线程执行时序。

![image-20220509142113921](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205091421963.png)

​		`JMM`可以确保读线程`C`至少能看到写线程A在构造函数中对`final`引用对象的成员域的写 入。即`C`至少能看到数组下标`0`的值为`1`。而写线程B对数组元素的写入，读线程`C`可能看得到， 也可能看不到。`JMM`不保证线程B的写入对读线程`C`可见，因为写线程`B`和读线程`C`之间存在数据竞争，此时的执行结果不可预知。

> 大致意思如下：只有在`intArray = new int[1];   intArray[0] = 1; `完成初始化后，才会去完成`FinalReferenceExample`构造函数的初始化，之后在`writerOne`方法中返回实例化后的对象被`obj`的引用。

## 为什么final引用不能从构造函数内“溢出”

​		前面我们提到过，写final域的重排序规则可以确保:在引用变量为任意线程可见之前，该 引用变量指向的对象的final域已经在构造函数中被正确初始化过了。其实，要得到这个效果， 还需要一个保证:在构造函数内部，不能让这个被构造对象的引用为其他线程所见，也就是对 象引用不能在构造函数中“逸出”。为了说明问题，让我们来看下面的示例代码。

```java
public class FinalReferenceEscapeExample {

    final int                          i;
    static FinalReferenceEscapeExample obj;

    public FinalReferenceEscapeExample() {
        i = 1; //1写final域
        obj = this; //2 this引用在此"逸出"
    }

    public static void writer() {
        new FinalReferenceEscapeExample();
    }

    public static void reader() {
        if (obj != null) { //3
            int temp = obj.i; //4
        }
    }
}
```

​		假设一个线程A执行writer()方法，另一个线程B执行reader()方法。这里的操作2使得对象 还未完成构造前就为线程B可见。即使这里的操作2是构造函数的最后一步，且在程序中操作2 排在操作1后面，执行read()方法的线程仍然可能无法看到final域被初始化后的值，因为这里的操作1和操作2之间可能被重排序。实际的执行时序可能如下图所示。

![image-20220509143915049](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202205091439092.png)

​		从图中可知：在构造函数为返回前，已经因为` obj = this;`引用的逸出，而导致`if (obj != null)`不为空，最后导致`int temp = obj.i;`获取到为初始化的值。

​		正常情况下：在构造函数返回前，被构造对象的引用不能为其他线程所见，因为此 时的`final`域可能还没有被初始化。在构造函数返回后，任意线程都将保证能看到`final`域正确初 始化之后的值。