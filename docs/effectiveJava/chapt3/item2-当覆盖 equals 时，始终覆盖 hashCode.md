## 前言

​		**在覆盖 equals 的类中，必须覆盖 hashCode。** 如果你没有这样做，你的类将违反 hashCode 的一般约定，这将阻止该类在 HashMap 和 HashSet 等集合中正常运行。

## Object规范的约定 

### 约定一

#### 定义

> 当在应用程序执行期间对对象重复调用 hashCode 方法时，它必须一致地返回相同的值，前提是不对 equals 比较中使用的信息进行修改。在不同的应用程序执行过程中，执行hashCode方法返回的值可以不同。

### 约定二

#### 定义

> 如果根据 `equals(Object)` 方法判断出两个对象是相等的，那么在两个对象上调用 hashCode 必须产生相同的整数结果。

#### 案列

##### 反例

定义类

```java
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = areaCode;
        this.prefix   = prefix;
        this.lineNum  = lineNum;
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof PhoneNumber))
            return false;
        PhoneNumber pn = (PhoneNumber)o;
        return pn.lineNum == lineNum && pn.prefix == prefix
                && pn.areaCode == areaCode;
    }

    //此处并没有重写hashCode方法
    // Remainder omitted - note that hashCode is REQUIRED (Item 11)!
}
```

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "Jenny");
m.get(new PhoneNumber(707, 867, 5309));
```

返回值

> null

结论

> PhoneNumber 类未能覆盖 hashCode，导致两个相等的实例具有不相等的 hash 代码，这违反了 hashCode 约定。因此，get 方法查找PhoneNumber的 hash 桶可能会与 put 方法存储PhoneNumber的 hash 桶不同。即使这两个实例碰巧 hash 到同一个 hash 桶上，get 方法也必定会返回 null，因为 HashMap 有一个优化，它缓存与每个条目相关联的 hash 码，如果 hash 码不匹配，就不会检查对象是否相等。

缓存与每个条目相关联的 hash 码

![image-20210729130840441](https://gitee.com/laoyouji1018/images/raw/master/img/20210729130841.png)

### 约定三

#### 定义

> 如果根据 `equals(Object)` 方法判断出两个对象不相等，则不一定要求在每个对象上调用 hashCode 时必须产生不同的结果。但是，程序员应该知道，为不相等的对象生成不同的结果可能会提高散列表的性能。

##### 案列

生成hashCode的规则：

```java
//要计算hash的类
public class Test {

    private String a;

    private float b;

    private long c;

    private double d;

    private Point point;


    private String[] strings;

    private Point[] points;

    private int[] ints;

    private double[] doubles;

    private float[] floats;

    private Double[] doublesTwo;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Test test = (Test) o;
        return Float.compare(test.b, b) == 0 && c == test.c && Double.compare(test.d, d) == 0 && a.equals(test.a) && point.equals(test.point) && Arrays.equals(strings, test.strings) && Arrays.equals(points, test.points) && Arrays.equals(ints, test.ints) && Arrays.equals(doubles, test.doubles) && Arrays.equals(floats, test.floats) && Arrays.equals(doublesTwo, test.doublesTwo);
    }
}
```

规则一

```java

    //对每个关键域的值进行hash计算
    @Override
    public int hashCode() {
        int result = a.hashCode();
        //如果字段是基本数据类型，计算 Type.hashCode(f)，其中 type 是与 f 类型对应的包装类。
        result = 31 * result + Float.hashCode(b);
        result = 31 * result + Long.hashCode(c);
        result = 31 * result + Double.hashCode(d);
        //如果字段是对象引用，并且该类的 equals 方法通过递归调用 equals 来比较字段，则递归调用字段上的 hashCode。如果需要更复杂的比较，则为该字段计算一个「规范表示」，并在规范表示上调用 hashCode。如果字段的值为空，则使用 0（或其他常数，但 0 是惯用的）。
        result = 31 * result + point.hashCode();
        //如果字段是一个数组，则将其视为每个重要元素都是一个单独的字段。也就是说，通过递归地应用这些规则计算每个重要元素的 hash 代码，并将每个步骤 2.b 的值组合起来。如果数组中没有重要元素，则使用常量，最好不是 0。如果所有元素都很重要，那么使用 Arrays.hashCode。
        result = 31 * result + Arrays.hashCode(strings);
        result = 31 * result + Arrays.hashCode(points);
        result = 31 * result + Arrays.hashCode(ints);
        result = 31 * result + Arrays.hashCode(doubles);
        result = 31 * result + Arrays.hashCode(floats);
        result = 31 * result + Arrays.hashCode(doublesTwo);
        return result;
    }
```

规则二

```java
@Override
    public int hashCode() {
        //通过Objects提供的hash方法父引用类型关键域和基本类型关键域进行hash计算
        int result = Objects.hash(a, b, c, d, point);
        //通过Arrays提供的hashCode方法对数组关键域进行hash计算
        result = 31 * result + Arrays.hashCode(strings);
        result = 31 * result + Arrays.hashCode(points);
        result = 31 * result + Arrays.hashCode(ints);
        result = 31 * result + Arrays.hashCode(doubles);
        result = 31 * result + Arrays.hashCode(floats);
        result = 31 * result + Arrays.hashCode(doublesTwo);
        return result;
    }
```

备注：

> 关键域：指的是前面equals方法中使用到的属性

> 衍生域：在散列码的计算过程中，可以把`衍生域`排除在外。换句话说，如果一个域的值可以根据参与计算的任何域计算出来，则可以排除这样的域参与计算。你必须排除不用于equals方法比较的任何字段，否则你可能会违反 hashCode 约定的第二个条款。

> `result = 31 * result + Arrays.hashCode(strings)`：计算散列的方法中，都对计算的散列进行乘法运算，这使散列值取决于字段的顺序，如果类有多个相似的字段，这样的乘法运算则会产生一个更好的 hash 函数。例如，如果字符串 hash 函数中省略了乘法，那么所有的姿势字母顺序不同的所有字符串都将会有相同的 hash 码。

> 为什么都是乘31：选择 31 是因为它是奇素数。如果是偶数，并且乘法运算溢出的话，信息就会丢失，因为与2相乘等价于移位运算。使用素数的好处不太明显，但它是传统的。31 的一个很好的特性是，可以用移位和减法来代替乘法，从而在某些体系结构上获得更好的性能：`31 * i == (i <<5) – i`。现代虚拟机自动进行这种优化。

不可变类的优化：

如果一个类是不可变的，并且计算 hash 代码的成本非常高，那么你可以考虑在对象中缓存 hash 代码，而不是在每次请求时重新计算它。如果你认为这种类型的大多数对象都将用作 hash 键，那么你应该在创建实例时计算 hash 代码。否则，你可能选择在第一次调用 hash 代码时延迟初始化 hash 代码。在一个延迟初始化的字段（Item-83）的情况下，需要一些注意来确保该类仍然是线程安全的。

```java
    @Override public int hashCode() {
        int result = hashCode;
        if (result == 0) {
            result = Short.hashCode(areaCode);
            result = 31 * result + Short.hashCode(prefix);
            result = 31 * result + Short.hashCode(lineNum);
            hashCode = result;
        }
        return result;
    }
```

### 约定四

#### 定义

> **不要试图从 hash 代码计算中排除重要字段，以提高性能。**虽然得到的 hash 函数可能运行得更快，但其糟糕的质量可能会将 hash 表的性能降低到无法使用的程度。如果发生这种情况， 散列函数将把所有这些实例映射到好书的散列码上。

### 约定五

#### 定义

> **不要为 hashCode 方法返回的值提供详细的规范，这样客户端就不能合理地依赖它。这（也）给了你更改它的灵活性。**

