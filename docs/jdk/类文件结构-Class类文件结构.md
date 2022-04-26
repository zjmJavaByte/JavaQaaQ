## Class类文件结构

​		**Class文件是一组以字节为基础单位的二进制流**，各个数据项目严格按照顺序紧凑地排列在文 件之中，中间没有添加任何分隔符，这使得整个Class文件中存储的内容几乎全部是程序运行的必要数据，没有空隙存在。当遇到需要占用8个字节以上空间的数据项时，则会按照高位在前的方式分割 成若干个8个字节进行存储。

​		**Class文件格式采用一种类似于C语言结构体的伪结构来存储数据，这种伪结构中只有两种数据类型:`无符号数`和`表`。**

- 无符号数：**属于基本的数据类型**，以`u1、u2、u4、u8`来分别代表1个字节、2个字节、4个字节和8个字节的无符号数，无符号数可以用来**描述数字**、**索引引用**、**数量值**或者按照**UTF-8编码构成字符串值**。
- 表：**是由多个无符号数或者其他表作为数据项构成的复合数据类型**，为了便于区分，所有表的命名都习惯性地以“_info”结尾。表用于描述有层次关系的复合结构的数据，整个Class文件本质上也可以视作是一张表，这张表由表6-1所示的数据项按严格顺序排列构成。

​		无论是无符号数还是表，当需要描述同一类型但数量不定的多个数据时，经常会使用一个前置的容量计数器加若干个连续的数据项的形式，这时候称这一系列连续的某一类型的数据为某一类型的“集合”，**例如下表中的接口集合、字段集合、方法集合、属性集合**。

​		以下是class文件格式总结表：

![image-20220422141858369](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204221418463.png)

​		任何一个Class文件都对应着唯一的一个类或接口的定义信息，但是反过来说，类或 接口并不一定都得定义在文件里(譬如类或接口也可以动态生成，直接送入类加载器中)

## 正文

​		通过简单的代码，进行各个结构的讲解，以下代码是通过java8进行编译：

```java
package interview;

public class TestClass {
    private int m;
    public strictfp int inc() {
        return m + 1;
    }
}
```

`		javap -verbose TestClass.class `反编译之后如下：

```java
Classfile /Users/zhujianmin/Documents/javaDoc/ideaProject/interview/src/main/java/interview/TestClass.class
  Last modified 2022-4-22; size 285 bytes
  MD5 checksum dbb92bea508369eda3fc312fc8de80f8
  Compiled from "TestClass.java"
public class interview.TestClass
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #4.#15         // java/lang/Object."<init>":()V
   #2 = Fieldref           #3.#16         // interview/TestClass.m:I
   #3 = Class              #17            // interview/TestClass
   #4 = Class              #18            // java/lang/Object
   #5 = Utf8               m
   #6 = Utf8               I
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               inc
  #12 = Utf8               ()I
  #13 = Utf8               SourceFile
  #14 = Utf8               TestClass.java
  #15 = NameAndType        #7:#8          // "<init>":()V
  #16 = NameAndType        #5:#6          // m:I
  #17 = Utf8               interview/TestClass
  #18 = Utf8               java/lang/Object
{
  public interview.TestClass();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0

  public strictfp int inc();
    descriptor: ()I
    flags: ACC_PUBLIC, ACC_STRICT
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: getfield      #2                  // Field m:I
         4: iconst_1
         5: iadd
         6: ireturn
      LineNumberTable:
        line 8: 0
}
SourceFile: "TestClass.java"
```

​		或者通过idea的`jclasslib`插件进行查看（具体使用清百度）

![image-20220422153038182](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204221530208.png)

​		或者在mac上通过`hex fiend` ,win使用`WinHex`查看十六进制

![image-20220422154155243](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204221541272.png)

​		接下来会根据这个十六进制进行一步步讲解。

### 魔数插

![image-20220422144544482](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204221445523.png)

​		通过文件表可知，魔数(Magic Number)占用开头的4个字节，**它的唯一作用是确定这个文件是否为 一个能被虚拟机接受的Class文件**。

​		从十六进制可以看到如下：

```java
//占用4个字节，一个字节在十六进制里面占两位
CAFEBABE
```

### class文件的版本

​		紧接着魔数的4个字节：Class文件的版本号

![image-20220422151843356](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204221518384.png)

```java
00000034
0000：小版本号
0034：大版本号
0034的十进制是52，52对应的是Java8
```

​		在我们的idea里面都有提示

![image-20220422152140735](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204221521778.png)

### 常量池

​		紧接着主、次版本号之后的是常量池入口，由于常量池中常量的数量是不固定的，所以在常量池的入口需要放置一项u2类型的数据，代表常量池容量计数值(constant_pool_count)

![image-20220422152631619](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204221526653.png)

```java
//十进制数值为19，所以常量池中有18项(为什么是18，而不是19，解释在下一段)，
//我们通过前面idea的jclasslib插件可验证为18
0013
```

​		在《深入理解Java虚拟机中》中的解释如下：**与Java中语言习惯不同，这个容量计数是从1而不是0开始 的，常量池容量为十六进制数0x0013，即十进制的19，这就代表常量池中有18项常量，索引值范围为1~18。在Class文件格式规范制定之时，设计者将第0项常量 空出来是有特殊考虑的，这样做的目的在于，如果后面某些指向常量池的索引值的数据在特定情况下 需要表达“不引用任何一个常量池项目”的含义，可以把索引值设置为0来表示。Class文件结构中只有 常量池的容量计数是从1开始，对于其他集合类型，包括接口索引集合、字段表集合、方法表集合等的 容量计数都与一般习惯相同，是从0开始。**

​		我们可以从Java中的`Object`对象的字节码可以查看到，其**父类索引**指向0，因为其没有父类。（至于父类索引，后面会讲到）

![image-20220422154026804](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204221540842.png)

​		我的理解就是，19项中，有一项是被0占用了，所以，就在十六进制所代表的数值基础上减1。

​		常量池中主要存放两大类常量：**字面量(Literal)和符号引用(Symbolic References)**

> 字面量：可以理解为实际值，int a = 8中的8 和 String a = "hello"中的hello都是字面量
>
> 符号引用：就是一个字符串，只要我们在代码中引用了一个非字面量的东西，不管它是变量还是常量，它都只是由一个字符串定义的符号，这个字符串存在常量池里，类加载的时候第一次加载到这个符号时，就会将这个符号引用（字符串）解析成直接引用（指针）
> 

​		**动态链接**：**Java代码在进行Javac编译的时候，并不像C和C++那样有“连接”这一步骤，而是在虚拟机加载Class 文件的时候进行动态连接**。也就是说，在Class文件中不会保存各个方法、字段最终在内存中的布局信息，这些字段、方法的符号引用不经过虚拟机在运行期转换的话是无法得到真正的内存入口地址，也就无法直接被虚拟机使用的。当虚拟机做类加载时，将会从常量池获得对应的符号
引用，再在类创建时或运行时解析、翻译到具体的内存地址之中。

​		常量池的17种数据类型的结构表



​		

**第一个常量项**		

​		从常量池容量计数值后开始第一个常量，它的标志位是

```java
0A   //十进制代表10
```

​		由常量数据类型表可知，这是一个`CONSTANT_Methodref_info`，表示的是类中方法的引用，其结构如下：

![image-20220422180234191](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204221802236.png)

```java
0004  //2个字节，十进制表示4，指向第4个常量项，指向方法的描述符
000F  //2个字节，十进制表示15，指向第15个常量项，指向名称和类型描述符
```

我们通过`jclasslib`插件也可以知道

![image-20220422181235030](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204221812082.png)

通过反编译的内容也可以知道

```java
Constant pool:
   #1 = Methodref          #4.#15         // java/lang/Object."<init>":()V
   #2 = Fieldref           #3.#16         // interview/TestClass.m:I
   #3 = Class              #17            // interview/TestClass
   #4 = Class              #18            // java/lang/Object
   #5 = Utf8               m
   #6 = Utf8               I
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               inc
  #12 = Utf8               ()I
  #13 = Utf8               SourceFile
  #14 = Utf8               TestClass.java
  #15 = NameAndType        #7:#8          // "<init>":()V
  #16 = NameAndType        #5:#6          // m:I
  #17 = Utf8               interview/TestClass
  #18 = Utf8               java/lang/Object

```

**第二个常量项**	

```java
09  //1个字节，十进制表示9
```

​		标志位是6，既`CONSTANT_Filedref_info`，字段符号引用

![image-20220422181029813](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204221810866.png)

```java
0003  //2个字节，十进制表示3，指向第3个常量项
0010  //2个字节，十进制表示16，指向第16个常量项
```

同样的通过`jclasslib`或者反编译也可以得知，就不一一展示了。

**第三个常量项**	

```java
07 //1个字节，十进制表示7
```

​		标志位是7，既`CONSTANT_Class_info`，类或者接口的符号引用

![image-20220422181748841](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204221817894.png)

```java
0011  //2个字节，十进制表示17，指向第17个常量项
```

​		我们根据反编译内容可知，第17位`CONSTANT_Utf8_info`，是一个`UTF-8`编码的字符串，代表类的权限定名称。顺便提一下，由于`Class`文件中方法、字段等都需要引用`CONSTANT_Utf8_info`型常量来描述名称，所以`CONSTANT_Utf8_info`型常量的最大长度也就是Java中方法、字段名的最大长度。而这里的 最大长度就是`length`的最大值，既u2类型能表达的最大值`65535`。所以J ava程序中如果定义了超过`64KB` 英文字符的变量或方法名，即使规则和全部字符都是合法的，也会无法编译。

**其他项就不一一举例了**，选中部分是剩下的敞亮项

![image-20220422185020776](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204221850809.png)

**也可以通过反编译查看：**

```java
Constant pool:
   #1 = Methodref          #4.#15         // java/lang/Object."<init>":()V
   #2 = Fieldref           #3.#16         // interview/TestClass.m:I
   #3 = Class              #17            // interview/TestClass
   #4 = Class              #18            // java/lang/Object
   #5 = Utf8               m
   #6 = Utf8               I
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               inc
  #12 = Utf8               ()I
  #13 = Utf8               SourceFile
  #14 = Utf8               TestClass.java
  #15 = NameAndType        #7:#8          // "<init>":()V
  #16 = NameAndType        #5:#6          // m:I
  #17 = Utf8               interview/TestClass
  #18 = Utf8               java/lang/Object
```

​	仔细看一下会发现，其中有些常量似乎从来 没有在代码中出现过，如`“ I”“ V”“ <init >”“ LineNumberTable”“ LocalVariableTable”`等，这些看起来在源代码中不存在的常量是哪里来的?

​		**这部分常量的确不来源于Java源代码，它们都是编译器自动生成的，会被后面即将讲到的字段表 (field_info)、方法表(met hod_info)、属性表(attribut e_info)所引用，它们将会被用来描述一些不方便使用“固定字节”进行表达的内容，譬如描述方法的返回值是什么，有几个参数，每个参数的类型是什么**。因为Java中的“类”是无穷无尽的，无法通过简单的无符号数来描述一个方法用到了什么类， 因此在描述方法的这些信息时，需要引用常量表中的符号引用进行表达。

### 访问标志

​		在常量池结束之后，紧接着的2个字节代表访问标志`(access_flags)`，这个标志用于识别一些类或者接口层次的访问信息，包括:这**个`Class`是类还是接口;是否定义为`public`类型;是否定义为`abstract `类型;如果是类的话，是否被声明为`final`**

![image-20220422184010429](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204221840494.png)

​		`TestClass`是一个普通`Java`类，不是接口、枚举、注解或者模块，被`public`关键字修饰但没有被声明为`final和abstract`，并且它使用了`JDK 1.2`之后的编译器进行编译，因此它的`ACC_PUBLIC、ACC_SUPER`标志应当为真，而`ACC_FINAL、ACC_INTERFACE、 ACC_ABSTRACT、ACC_SYNTHETIC、ACC_ANNOTATION、ACC_ENUM、ACC_MODULE`这七 个标志应当为假

```java
0x0021 = 0x0001|0x0020  ACC_PUBLIC | ACC_SUPER
```

### 类索引、父类索引与接口索引集合

​		类索引`(this_class)`和父类索引`(super_class)`都是一个`u2`类型的数据，而接口索引集合`(interfaces)`是一组u2类型的数据的集合，`Class`文件中由这三项数据来确定该类型的继承关系。类索引用于确定这个类的全限定名，父类索引用于确定这个类的父类的全限定名，接口索引集合就用来描述这个类实现了哪些接 口，这些被实现的接口将按`implements`关键字(如果这个`Class`文件表示的是一个接口，则应当是`extends`关键字)后的接口顺序从左到右排列在接口索引集合中。

​		类索引、父类索引和接口索引集合都按顺序排列在访问标志之后，类索引和父类索引用两个u2类 型的索引值表示，它们各自指向一个类型为CONSTANT_Class_info的类描述符常量，通过 CONSTANT_Class_info类型的常量中的索引值可以找到定义在CONSTANT_Utf8_info类型的常量中的 全限定名字符串。

```java
0003 //第三个常量 #3
0004 //第四个常量 #4
```

​		对于接口索引集合，入口的第一项u2类型的数据为接口计数器(int erfaces _count )，表示索引表 的容量。

```java
0000 //接口数量为0
```

### 字段表集合

​		**字段表(`field_info`)用于描述接口或者类中声明的变量。Java语言中的“字段”(Field)包括类级变量以及实例级变量，但不包括在方法内部声明的局部变量。**读者可以回忆一下在Java语言中描述一个 字段可以包含哪些信息。

​		**字段包括的修饰符：**字段的作用域(public、privat e、prot ect ed修饰符)、是实例变量还是类变量(static修饰符)、可变性(final)、并发可见性(volatile修饰符，是否强制从主内存读写)、可否被序列化(transient修饰符)、字段数据类型(基本类型、对象、数组)、 字段名称。上述这些信息中，各个修饰符都是布尔值，要么有某个修饰符，要么没有，很适合使用标志位来表示。而字段叫做什么名字、字段被定义为什么数据类型，这些都是无法固定的，只能引用常量池中的常量来描述。

![image-20220423120921332](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204231209395.png)

首先是`u2`类型的字段数量

```java
0001  //代表字段只有一个
```

其次是一个`filed_info`,既字段表，其结构如下：

![image-20220423121110634](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204231211677.png)

`u2`的`access_flags`，访问标志，选项如下

![image-20220423121250018](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204231212066.png)

```java
0002  //代表其实private修饰
```

`u2`的`name_index`，字段的简单名称索引项，指向常量池

> ​		全限定名和简单名称很好理解，以用例代码，`interview/TestClass `是这 个类的全限定名，仅仅是把类全名中的“ .”替换成了“ /”而已，为了使连续的多个全限定名之间不产生混淆，在使用时最后一般会加入一个“;”号表示全限定名结束。简单名称则就是指没有类型和参数修饰 的方法或者字段名称，这个类中的`inc()`方法和`m`字段的简单名称分别就是`inc和 m`。

```java
0005  //由反编译的结果可知，其实是字母 m 
```

`u2`的`descriptor_index`，字段或者方法的描述符，指向常量池。

> ​		描述符的作用是用来描述字段 的数据类型、方法的参数列表(包括数量、类型以及顺序)和返回值。根据描述符规则，基本数据类 型(by te、char、double、float、int、long、short、boolean)以及代表无返回值的void类型都用一个大 写字符来表示，而对象类型则用字符L加对象的全限定名来表示，

```java
0006 //由反编译的结果可知，其实是字母 I
```

描述符的表示字符含义如下：

![image-20220423122946596](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204231229638.png)

> void类型在《Java虚拟机规范》之中单独列出为“ VoidDescriptor”，笔者为了结构统一，将其列 在基本数据类型中一起描述。
>
> 
>
> **数组类型描述符**：每一维度将使用一个前置的`[`字符来描述，如一个定义为` java.lang.String[][]`类型 的二维数组将被记录成` [[Ljava/lang/St ring;`，一个整型数组` int []`将被记录成`[`
>
> **方法描述符：**用描述符来描述方法时，按照先参数列表、后返回值的顺序描述，参数列表按照参数的严格顺序 放在一组小括号`()`之内。如方法`void inc()`的描述符为`()V`，方法`java.lang.String toString()`的描述符 为`()Ljava/lang/String;`，方法`int indexOf(char[]source，int sourceOffset，int sourceCount，char[]target， int targetOffset，int targetCount，int fromIndex)`的描述符为`([CII[CIII)I`。

由上可知该字段为今本类型 `int`，所以得出结论该字段源代码为：`private int m`。

**注意事项**

- 字段表集合中不会列出从父类或者父接口中继承而来的字段，但有可能出现原本Java代码之中不存在的字段，譬如在内部类中为了保持对外部类的访问性，编译器就会自动添加指向外部类实例的字 段。

- 在Java语言中字段是无法重载的，两个字段的数据类型、修饰符不管是否相同，都必须使 用不一样的名称，但是对于Class文件格式来讲，只要两个字段的描述符不是完全相同，那字段重名就 是合法的。

### 方法表集合

​		Class文件存储 格式中对方法的描述与对字段的描述采用了几乎完全一致的方式，方法表的结构如同字段表一样，依次包括`访问标志(access_flags)、名称索引(name_index)、描述符索引(descrip tor_index)、属性表集合(attributes)`几项。

![image-20220423125215801](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204231252857.png)

首先是`u2`类型的字段数量

```java
0002  //代表方法有2个
```

其次是一个`method_info`,既方法表表，其结构如下：

![image-20220423125058815](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204231250862.png)

`u2`类型的`access_flags`

![image-20220423125552472](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204231255517.png)

```java
0001  //代表该方法是 public
```

`u2`类型的`name_index`名称索引

```java
0007 //实例构造器<init>方法
```

`u2`的`descriptor_index`描述符索引

```java
0008  //()V
```

`u2`的`attributes_count`属性表计数器

```java
0001 //表示此方法的属性表集合有1项属性
```

`attributes_info`属性表集合，其结构为

![image-20220423160844432](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204231608505.png)

`attributes_name_index`

```java
0009  //属性名称,对应常量为“ Code”
```

`attributes_length`

```java
0000001D   //所以长度为29
```

> **实例构造器：**编辑期间自动生成的无参构造函数
>
> **类构造器：**静态代码块
>
> **重载：** **要重载(Overload)一个方法，除了要与原方法具有相同的简单名称之外，还要求必须拥有一个与原方法不同的特征签名**。特征签名是指一个方法中各个参数在常量池中的字段符号引用的集合，也正是因为返回值不会包含在特征签名之中，所以Java语言里面是无法仅仅依靠返回值 的不同来对一个已有方法进行重载的。
>
> **特征签名：**Java代码的方法特征签名只包括方法名称、参数 顺序及参数类型，而字节码的特征签名还包括方法返回值以及受查异常表

### 属性表集合

​		对于每一个属性，它的名称都要从常量池中引用一个CONSTANT_Utf8_info类型的常量来表示， 而属性值的结构则是完全自定义的，只需要通过一个u4的长度属性去说明属性值所占用的位数即可。 一个符合规则的属性表应该满足下表中所定义的结构。

![image-20220423164143720](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204231641771.png)

#### Code属性

​		**方法的定义可以通过访问标志、名称索引、描述符索引来表达清楚，方法里的Java代码，经过Javac编译器编译成字节码指令之 后，存放在方法属性表集合中一个名为“Code”的属性里面**。

​		Code属性是Class文件中最重要的一个属性，如果把一个Java程序中的信息分为代码(Code，方法体里面的Java代码)和元数据(Metadata，包括类、字段、方法定义及其他信息)两部分，那么在整 个Class文件里，Code属性用于描述代码，所有的其他数据项目都用于描述元数据。

##### 表结构如下

![image-20220423164829123](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204231648175.png)

**attribute_name_index**:属性名称索引，是一项指向CONSTANT_Utf8_info型常量的索引，此常量值固定为“Code”

**attribute_length**：指示了属性值的长度，由于属性名称索引与属性长度一共为 6个字节，所以属性值的长度固定为整个属性表长度减去6个字节

**max_stack**：代表了[操作数栈](https://github.com/zjmJavaByte/JavaQaaQ/blob/master/docs/jdk/%E6%93%8D%E4%BD%9C%E6%95%B0%E6%A0%88.md)(Operand Stack)深度的最大值。在方法执行的任意时刻，操作数栈都 不会超过这个深度。虚拟机运行的时候需要根据这个值来分配栈帧(Stack Frame)中的操作栈深度。

**max_locals**:代表了[局部变量表](https://github.com/zjmJavaByte/JavaQaaQ/blob/master/docs/jdk/%E5%B1%80%E9%83%A8%E5%8F%98%E9%87%8F%E8%A1%A8.md)所需的存储空间

**code_length和code**：code_length和code用来存储Java源程序编译后生成的字节码指令。code_length代表字节码长度， code是用于存储字节码指令的一系列字节流(具体的[虚拟机字节码指令表](https://blog.csdn.net/qq_15103197/article/details/122508442))。

> **方法体的大小是有限制的**：`code_length`代表字节码指令的个数，其长度为`u4`，而方法提编译后就是字节码指令流，所以当超过指定的长度之后，编译器就会编译失败。理论上最大值可以达到2的32次幂，但是《Java虚拟机规范》中明确限制了一个方法不允许超过`65535`条字节码指令，即它 实际只使用了`u2`的长度，如果超过这个限制，`Javac`编译器就会拒绝编译

由上一节可知，我们读到`attribute_length`为5，接下来就是`max_stack`

```java
0001  //操作数栈为1
```

`max_locals`

```java
0001  //局部变量表为1
```

`code_length`

```java
00000005  //长度为5个字节
```

`code`

```java
2A B7 0001 B1
		2A：读入2A，查表得0x2A对应的指令为aload_0，这个指令的含义是将第0个变量槽中为reference类 型的本地变量推送到操作数栈顶。
		B7：读入B7，查表得0xB7对应的指令为invokesp ecial，这条指令的作用是以栈顶的reference类型的 数据所指向的对象作为方法接收者，调用此对象的实例构造器方法、p rivat e方法或者它的父类的方法。这个方法有一个u2类型的参数说明具体调用哪一个方法，它指向常量池中的一个 CONSTANT_M ethodref_info类型常量，即此方法的符号引用。
		0001：读入0001，这是invokesp ecial指令的参数，代表一个符号引用，查常量池得0x000A对应的常量 为实例构造器“<init>()”方法的符号引用。
		B1：读入B1，查表得0xB1对应的指令为ret urn，含义是从方法的返回，并且返回值为void。这条指 令执行后，当前方法正常结束。
```

​		通过idea插件`jclasslib`也可以得出上述结果

![image-20220423173301084](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204231733137.png)

或者通过反编译` javap -verbose TestClass.class`

```java
.........
{
  public interview.TestClass();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0

  public int inc();
    descriptor: ()I
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: getfield      #2                  // Field m:I
         4: iconst_1
         5: iadd
         6: ireturn
      LineNumberTable:
        line 8: 0
}
.........
```

> 大家是否注意到这里的两个方法的`args_size`既参数都有一个，结论可以查看文章 [局部变量表](https://github.com/zjmJavaByte/JavaQaaQ/blob/master/docs/jdk/%E5%B1%80%E9%83%A8%E5%8F%98%E9%87%8F%E8%A1%A8.md)

​		我们可以看到在`code`属性表里面还有一个异常表，异常表对于Code 属性来说并不是必须存在的，如前面的代码就没有。

`exception_table`

​		异常表包含四个字段，这些字段的含义为:如果当字节码从第`start_pc`行到第`end_pc`行之间(不含第`end_pc`行)出现了类型为`catch_type`或者其子类的异常 (`catch_type`为指向一个`CONSTANT_Class_info`型常量的索引)，则转到第handler_pc行继续处理。当 cat ch_t y p e的值为0时，代表任意异常情况都需要转到handler_p c处进行处理。

![image-20220423175148281](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204231751339.png)

案列代码如下(猜测下有异常和无异常情况的`x`值)：

```java
// Java源码
    public int inc() {
        int x;
        try {
            x = 1;
            return x;
        } catch (Exception e) {
            x = 2;
            return x;
        } finally {
            x = 3;
        }
    }
// TODO 未完待续
```

 编译后的ByteCode字节码及异常表

```java
 0 iconst_1  //加载常量 x=1 到操作数栈
 1 istore_1  //将x=1存储到局部变量表slot为1的位置
 2 iload_1	 //加载slot为1的局部变量x=1到操作数栈
 3 istore_2  //将x=1复制一份副本到slot为2的位置
 4 iconst_3		//加载finally块中的x=3到操作数栈
 5 istore_1		//将x=3存储到slot为1的位置，既覆盖x=1
 6 iload_2		//加载副本x=2到操作数栈
 7 ireturn		//返回x=2的值
 8 astore_2		//给catch中定义的 Exception e 赋值，存储在slot为2中
 9 iconst_2		//加载x=2到操作数栈
10 istore_1		//存储x=2到slot为1的位置，覆盖x=3
11 iload_1		//加载x=2到操作数栈
12 istore_3		//存储x=2的副本到slot为3的位置
13 iconst_3		//加载finaly块中的x=3
14 istore_1		//存储到slot为1的位置，覆盖之前的x=1
15 iload_3		//加载slot为3，x=2到操作数栈
16 ireturn		//返回x=2
17 astore 4		//如果出现了不属于java.lang.Exception及其子类的异常才会走到这里
19 iconst_3		//加载x=3到操作数栈
20 istore_1		//存储到局部变量表slot为1的位置
21 aload 4		//加载astore 4存储的异常到操作数栈
23 athrow			//抛出异常
   
Exception table:
         from    to  target type
             0     4     8   Class java/lang/Exception
             0     4    17   any
             8    13    17   any
            17    19    17   any

```

#### Exceptions属性

​		这里的`Exceptions`属性是在方法表中与`Code`属性平级的一项属性，读者不要与前面刚刚讲解完的异常表产生混淆。**`Exceptions`属性的作用是列举出方法中可能抛出的受查异常`(Checked Excepitons)`，也就是方法描述时在`throws`关键字后面列举的异常。**

**表结构**

![image-20220424100554303](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204241005847.png)

​		此属性中的`number_of_exceptions`项表示方法可能抛出`number_of_excep tions`种受查异常，每一种受 查异常使用一个`exception_index_table`项表示;`exception_index_table`是一个指向常量池中` CONSTANT_Class_info`型常量的索引，代表了该受查异常的类型。

```java
package interview;

public class TestClass {
    private int m;

    public TestClass() {
    }
    public int inc() throws NumberFormatException {
        return this.m + 1;
    }
}
```

通过idea的`jclasslib`插件

![image-20220424101212767](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204241012800.png)

#### LineNumberTable属性

​		**`LineNumberTable`属性用于描述`Java`源码行号与字节码行号(字节码的偏移量)之间的对应关系。** 它并不是运行时必需的属性，但默认会生成到`Class`文件之中，可以在`Javac`中使用`-g:none`或`-g:lines` 选项来取消或要求生成这项信息。如果选择不生成`LineNumberTable`属性，对程序运行产生的最主要影响就是当抛出异常时，堆栈中将不会显示出错的行号，并且在调试程序的时候，也无法按照源码行来 设置断点。

![image-20220424101726689](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204241017736.png)

​		`line_number_table`是一个数量为`line_number_table_length`、类型为`line_number_info`的集合，`line_number_info`表包含`start_pc`和`line_number`两个`u2`类型的数据项，前者是字节码行号，后者是`Java`源码行号。

![image-20220424101845114](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204241018164.png)

#### LocalVariableTable及LocalVariableTypeTable属性

​		**`LocalVariableTable`属性用于描述栈帧中局部变量表的变量与`Java`源码中定义的变量之间的关系**，它也不是运行时必需的属性，但默认会生成到`Class`文件之中，可以在`Javac`中使用`-g:none`或`-g:vars`选项来取消或要求生成这项信息。

![image-20220424103216137](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204241032170.png)

![image-20220424103204687](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204241032730.png)

#### SourceFile及SourceDebugExtension属性

​		**SourceFile属性用于记录生成这个Class文件的源码文件名称**。这个属性也是可选的，可以使用Javac 的-g:none或-g:source选项来关闭或要求生成这项信息。

**SourceFile属性表结构**

![image-20220424103433033](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204241034087.png)

![image-20220424103701852](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204241037910.png)

#### ConstantValue属性

//TODO 类构造器，实例构造器 （前端编译优化有讲解）

​		**`ConstantValue`属性的作用是通知虚拟机自动为静态变量赋值**。只有被`stat ic`关键字修饰的变量(类变量)才可以使用这项属性。

**非`static`变量**

```java
package interview;

public class TestClass {
    private int m;
}
```

赋值是在实例构造器`<init>()`方法中进行的

**`static`变量**

​		如果同时使用`final`和`static`来修饰一个变量(按照习惯，这里称“静态常量”更贴切)，并且这个变量的数据类型是**基本类型**或者`java.lang.String`的话，字段表中就将会生成`ConstantValue`属性来进行初始化

```java
package interview;

public class TestClass {
    private final static String A = "123";
}		
```

​		如果这个变量没有被`final`修饰，或者并非基本类型及字符串，方法表中则将会选择在`<clinit>()`方法中进行初始化。

```java
package interview;

public class TestClass {
    private static String A = "123";
}	
```





**ConstantValue属性结构**

![image-20220424111506857](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204241115894.png)

![image-20220424111424196](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204241114253.png)

​		`constantvalue_index`数据项代表了常量池中一个字面量常量的引用，根据字段类型的不同，字面 量可以是`CONSTANT_Long_info、CONSTANT_Float_info、CONSTANT_Double_info、 CONSTANT_Integer_info和CONSTANT_String_info`常量中的一种。

#### InnerClasses属性

​		**`InnerClasses`属性用于记录内部类与宿主类之间的关联。**如果一个类中定义了内部类，那编译器将会为它以及它所包含的内部类生成`InnerClasses`属性。

**InnerClasses属性结构**

![image-20220424112123168](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204241121217.png)

**inner_classes_info表的结构**

![image-20220424112151498](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204241122169.png)

![image-20220424112101102](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204241121154.png)

​		`inner_class_access_flags`是内部类的访问标志

![image-20220424112323400](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204241123460.png)

#### Deprecated及Synthetic属性

​		**`Deprecated`和`Synthetic`两个属性都属于标志类型的布尔属性，只存在有和没有的区别，没有属性值的概念。**

​		`Deprecated`属性用于表示某个类、字段或者方法，已经被程序作者定为不再推荐使用，它可以通过代码中使用`@deprecat ed`注解进行设置。

​		`Synthetic`属性代表此字段或者方法并不是由Java源码直接产生的，而是由编译器自行添加的，在 `JDK 5`之后，标识一个类、字段或者方法是编译器自动产生的，也可以设置它们访问标志中的 `ACC_SYNTHETIC`标志位。编译器通过生成一些在源代码中不存在的`Synthetic`方法、字段甚至是整个 类的方式，实现了越权访问(越过`private`修饰器)或其他绕开了语言限制的功能，这可以算是一种早 期优化的技巧，其中最典型的例子就是枚举类中自动生成的枚举元素数组和嵌套类的桥接方法 `(Bridge Method)`。所有由不属于用户代码产生的类、方法及字段都应当至少设置`Synthetic`属性或者 `ACC_SYNTHETIC`标志位中的一项，唯一的例外是实例构造器`<init>()`方法和类构造器`<clinit>()`方 法。

**Deprecated及Synthetic属性结构**

![image-20220424113116014](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204241131076.png)

​		其中`attribute_length`数据项的值必须为`0x00000000`，因为没有任何属性值需要设置。

#### StackMapTable属性

//TODO 字节码验证部分有讲解

​		`StackMapTable`属性在JDK 6增加到Class文件规范之中，它是一个相当复杂的变长属性，位于`Code `属性的属性表中。这个属性会在虚拟机类加载的字节码验证阶段被新类型检查验证器(`Type Checker`)使用)，目的在于代替以前比较消耗性能的基于数据流分析的 类型推导验证器。

#### Signature属性

//TODO 编译器优化有讲解

​		它是一个可选的定长属性，可以出现于类、字段 表和方法表结构的属性表中。在JDK 5里面大幅增强了Java语言的语法，在此之后，任何类、接口、初 始化方法或成员的泛型签名如果包含了类型变量(`Type Variable`)或参数化类型(`Parameterized Type`)，则`Signature`属性会为它记录泛型签名信息。

```java
package interview;

public class TestClass<T> {
    public int inc(T t)  {
        return  1;
    }
}
```

**Signature属性结构**

![image-20220424114724818](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204241147882.png)

![image-20220424114629358](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204241146418.png)

#### BootstrapMethods属性

// TODO` InovkeDynamic`指令的运作原理

​		**它是一个复杂的变长属性，位于类文件的属性表中。这个属性用于保存`invokedynamic`指令引用的引导方法限定符。**

**BootstrapMethods属性结构**

![image-20220424115542852](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204241155924.png)

**bootstrap_method属性结构**

![image-20220424115555926](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204241155987.png)

#### MethodParameters属性

​		`MethodParameters`是在`JDK 8`时新加入到`Class`文件格式中的，它是一个用在方法表中的变长属性。`MethodParameters`的作用是记录方法的各个形参名称和信息。

​		在`LocalVariableTable`中，记录的是描述栈帧中局部变量表的变量与`Java`源码中定义的变量之间的关系，但是抽象方法和接口方法，是理所当然地可以不存在方法体的，对于方法签名来说，还是没有找到一个统一完整的保留方法参数名称的地方。所以在JDK 8中新增的这个属性，使得编译器可以 (编译时加上`-parameters`参数)将方法名称也写进Class文件中，而且MethodParameters是方法表的属 性，与Code属性平级的，可以运行时通过反射API获取。

**MethodParameters属性结构**

![image-20220424120821446](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204241208483.png)

**parameter属性结构**

![image-20220424120855090](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204241208124.png)

![image-20220424120738649](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204241207722.png)

access_flags是参数的状态指示器，它可以包含以下三种状态中的一种或多种:

- `0x0010(ACC_FINAL)`:表示该参数被final修饰。

- `0x1000(ACC_SYNTHETIC)`:表示该参数并未出现在源文件中，是编译器自动生成的。

- `0x8000(ACC_MANDATED)`:表示该参数是在源文件中隐式定义的。Java语言中的典型场景是 this关键字。

#### 模块化相关属性



#### 运行时注解相关属性

​		`RuntimeVisibleAnnotations`是一个变长属性，它记录了类、字段或方法的声明上记录运行时可见注 解，当我们使用反射API来获取类、字段或方法上的注解时，返回值就是通过这个属性来取到的。

**案列代码**

```java
package interview;

import java.beans.Transient;

public class TestClass {

    @Deprecated
    @Transient
    public  int inc(int a){
        return a;
    }
}
```

**RuntimeVisibleAnnotations属性结构**

![image-20220424121656371](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204241216432.png)

**annotation属性结构**

![image-20220424121821315](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204241218380.png)

​		`num_element_value_pairs`是`element_value_pairs`数组的计数器，`element_value_pairs`中每 个元素都是一个键值对，代表该注解的参数和值。

![image-20220424122025081](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204241220144.png)