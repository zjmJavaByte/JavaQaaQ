[TOC]

## JDK之JVM中Java对象的头部占多少byte

 先做个铺垫：

- ​    在32位机器上word size是32bits，CPU一次性处理32bits，在64位机器上word size是64bits，CPU一次性处理64bits。
- ​    Data bus size, instruction size, address size are usually multiples of the word size，这句参考自[Stackoverflow](https://stackoverflow.com/questions/19821103/what-does-it-mean-by-word-size-in-computer)。  

### Stackoverflow上看到的Java对象头部mark word和kclass pointer的大小

[  从Stackoverflow上](https://stackoverflow.com/questions/26357186/what-is-in-java-object-header)看到，Java对象头部有一个**mark word**和一个**klass pointer，**

-  mark word：32bits architectures上，mark word占32bits，64bits architectures上，mark word占64bits；
-   kclass pointer：32bits architectures上，kclass pointer占32bits，64bits architectures上，kclass pointer占64bits，但也可能是32bits，原话是这样"the **klass pointer** has word size on `32 bit` architectures. On `64 bit` architectures the klass pointer either has word size, but can also have `4 byte` if the heap addresses can be encoded in these `4 bytes`"。

### 验证上面的说法

  我们来验证下。注意：我电脑上装的JDK1.8，64位的

![image-20220413115532230](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204131155008.png)

#### 事先准备

maven项目需要导入jar包

```xml
<!-- https://mvnrepository.com/artifact/org.openjdk.jol/jol-core -->
        <dependency>
            <groupId>org.openjdk.jol</groupId>
            <artifactId>jol-core</artifactId>
            <version>0.16</version>
        </dependency>
```

#### 验证Java对象的头部占用byte数

```java
package interview;

import org.openjdk.jol.info.ClassLayout;
import org.openjdk.jol.info.GraphLayout;
import org.openjdk.jol.vm.VM;

public class Obj {
    public static void main(String[] args) throws Exception {
        System.out.println(VM.current().details());
        final A a = new A();
        System.out.println("=======================");
        String s = ClassLayout.parseInstance(a).toPrintable();
        System.out.println("查看对象内部信息--->>>"+s);
        System.out.println("=======================");
        String s1 = GraphLayout.parseInstance(a).toPrintable();
        System.out.println("查看对象外部信息：包括引用的对象--->>>"+s1);
        System.out.println("=======================");
        long l = GraphLayout.parseInstance(a).totalSize();
        System.out.println("查看对象占用空间总大小--->>>"+l);
    }
    public static class A {
        // no fields
    }
}
```

测试结果如下：

![image-20220413120400189](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204131204222.png)

**object header: mark**：`mark word` 占用8byte，也就是64bit

**object header: class**：`kclass point`占用4byte，也就是32bit

**object alignment gap**：`对齐填充`占用4byte，也就是32bit

我的电脑是64位，对象占用16byte，所以大小总的占用大小是word size的倍数。

#### 验证Java数组对象的头部占用byte数

```java
package interview;


import org.openjdk.jol.info.ClassLayout;
import org.openjdk.jol.vm.VM;

public class Obj {

    public static void main(String[] args) throws Exception {
        System.out.println(VM.current().details());
        A[] a = new A[2];
        System.out.println("=======================");
        ClassLayout layout = ClassLayout.parseInstance(a);
        System.out.println(layout);
        System.out.println("=======================");
        String s = ClassLayout.parseInstance(a).toPrintable();
        System.out.println("查看对象内部信息--->>>"+s);
    }
    public static class A {
        // no fields
    }
}


```

测试结果如下：

![image-20220413122301894](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204131223923.png)

**object header: mark**：`mark word` 占用8byte，也就是64bit

**object header: class**：`kclass point`占用4byte，也就是32bit

**array length**：数组长度占用4byte

**alignment/padding gap**：`对齐填充`占用4byte，也就是32bit

**interview.Obj$A Obj$A;.<elements>**    ： 数组new A[2]的长度是2，每个下标处占4bytes，这个类似C语言中的指针。

所以可以看到数组和普通的Java对象头部是有区别的。

[参考文章](https://cloud.tencent.com/developer/article/1413543)