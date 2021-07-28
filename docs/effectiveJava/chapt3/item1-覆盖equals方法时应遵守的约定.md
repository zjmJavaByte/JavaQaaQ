## 前言

​		虽然 `Object` 是一个具体的类，但它主要是为继承而设计的。它的所有非 `final` 方法（`equals、hashCode、toString、clone 和 finalize`）都有清晰的通用约定（` general contracts`），因为它们被设计为被子类重写。任何类要重写这些方法时，都有义务去遵从它们的通用约定；如果不这样做，将会阻止其他依赖于约定的类 (例如 `HashMap` 和 `HashSet`) 与此类一起正常工作。

## 覆盖 equals 方法时应遵守的约定

### 什么时候不需要覆盖

- **类的每个实例本质上都是唯一的。** 对于像 Thread 这样表示活动实体类而不是值类来说也是如此。Object 提供的 equals 实现对于这些类具有完全正确的行为。

```java
//活动实体类而不是值类：比较是不是同一个对象，而不去管属性值是不是一样；以下是Object中的equal方法
public boolean equals(Object obj) {
        return this == obj;
}
```

- **该类不需要提供「逻辑相等」测试。** 例如，`java.util.regex.Pattern` 可以覆盖 equals 来检查两个 Pattern 实例是否表示完全相同的正则表达式，但设计人员认为客户端不需要或不需要这个功能。在这种情况下，从 Object 继承的 equals 实现是理想的。

- **类是私有的或包私有的，并且你确信它的 equals 方法永远不会被调用。** 如果你非常想要规避风险，你可以覆盖 equals 方法，以确保它不会意外调用：

```java
@Override
public boolean equals(Object o) {
    throw new AssertionError(); // Method is never called
}
```

- **最多只存在一个对象的类、枚举类型**

### 什么时候应该覆盖

​		当一个类有一个逻辑相等的概念，而这个概念不同于仅判断对象的同一性（相同对象的引用），并且超类还没有覆盖 equals。对于值类通常是这样。值类只是表示值的类，例如 Integer 或 String。使用 equals 方法比较引用和值对象的程序员希望发现它们在逻辑上是否等价，而不是它们是否引用相同的对象。

#### 覆盖equal方法遵守的约定

- 自反性：对于任何非空的引用值 x，`x.equals(x)` 必须返回 true。

- 对称性：对于任何非空引用值 x 和 y，当且仅当`x.equals(y)` 为true时， `y.equals(x)` 必须返回 true 。

违法上述约定的代码

```java
// Broken - violates symmetry!
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
}

// Broken - violates symmetry!
@Override
public boolean equals(Object o) {
    if (o instanceof CaseInsensitiveString)
    return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);

    if (o instanceof String) // One-way interoperability!
        return s.equalsIgnoreCase((String) o);

    return false;
    } ... // Remainder omitted
}

```

假设我们有一个不区分大小写的字符串和一个普通字符串

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";
```

正如预期的那样，`cis.equals(s)` 返回 true。问题是，虽然 `CaseInsensitiveString `中的 `equals `方法知道普通字符串，但是 `String` 中的 `equals` 方法对不区分大小写的字符串不知情。因此，`s.equals(cis)` 返回 false，这明显违反了对称性。假设你将不区分大小写的字符串放入集合中：

```java
// Demonstration of the problem (Page 40)
    public static void main(String[] args) {
        CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
        String s = "polish";

        List<CaseInsensitiveString> list = new ArrayList<>();
        list.add(cis);

        System.out.println(list.contains(s));
    }
```

当前的` OpenJDK` 实现中`list.contains(s)`返回false

```java
false
```

修改equal方法

```java
    // Fixed equals method (Page 40)
@Override public boolean equals(Object o) {
      return o instanceof CaseInsensitiveString &&
          ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```

- 传递性：对于任何非空的引用值 x, y, z，如果 `x.equals(y)` 返回 true，`y.equals(z)` 返回 true，那么 `x.equals(z)` 必须返回 true。

让我们从一个简单的不可变二维整数点类开始：

```java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point)o;
        return p.x == x && p.y == y;
    }
    ... // Remainder omitted
}

```

子类添加了一条影响 equals 比较的信息

```java
public class ColorPoint extends Point {
    private final Color color;//添加的字段属性

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
    ... // Remainder omitted
}

```

如果不覆盖父类Point 的equal方法

```java
public static void main(String[] args) {
        Point p = new Point(1, 2);
        ColorPoint cp = new ColorPoint(1, 2, Color.RED);
        System.out.println(p.equals(cp) + " " + cp.equals(p)); //比较的是x,y两个值是否相等
}
```

> true  true

如果覆盖父类Point 的equal方法，并且添加上color

```java

@Override
public boolean equals(Object o) {
    if (!(o instanceof ColorPoint))
        return false;
    return super.equals(o) && ((ColorPoint) o).color == color;
}
```

```java
public static void main(String[] args) {
        Point p = new Point(1, 2);
        ColorPoint cp = new ColorPoint(1, 2, Color.RED);
        System.out.println(p.equals(cp) + " " + cp.equals(p)); 
}
```

> true  false

从以上结果发现违反了对称性

为了达到对称性，让`ColorPoint.equal()`方法在实现混合比较时（父子类比较），忽略颜色

```java
//修改ColorPoint中的equal方法
@Override
public boolean equals(Object o) {
    if (!(o instanceof Point))
        return false;

    // If o is a normal Point, do a color-blind comparison
    if (!(o instanceof ColorPoint))
        return o.equals(this);

    // o is a ColorPoint; do a full comparison
    return super.equals(o) && ((ColorPoint) o).color == color;
}

```

```java
public static void main(String[] args) {
        // First equals function violates symmetry (Page 42)
        Point p = new Point(1, 2);
        ColorPoint cp = new ColorPoint(1, 2, Color.RED);
        System.out.println(p.equals(cp) + " " + cp.equals(p));

        // Second equals function violates transitivity (Page 42)
        ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
        Point p2 = new Point(1, 2);
        ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
        System.out.printf("%s %s %s%n",
                          p1.equals(p2), p2.equals(p3), p1.equals(p3));
    }
```

> true true
> true true false

虽然经过上面后保证了对称性，但是却违反了传递性。

并且，这种方法会导致无限的递归：假设有两个点的子类，比如 `ColorPoint` 和 `SmellPoint`，每个都使用这种 `equals` 方法。然后调用 `myColorPoint.equals(mySmellPoint)` 会抛出 `StackOverflowError`。

> 那么解决方案是什么？这是面向对象语言中等价关系的一个基本问题。**除非你愿意放弃面向对象的抽象优点，否则无法继承一个可实例化的类并添加一个值组件，同时保留 equals 约定。**

通过在Point中的 equals 方法中使用 `getClass` 测试来代替 `instanceof` 测试，可以扩展实例化的类并添加一个值组件，并且来保持 equals 约定：

```java
// Broken - violates Liskov substitution principle (page 43)
@Override
public boolean equals(Object o) {

    if (o == null || o.getClass() != getClass())
        return false;

    Point p = (Point) o;
    return p.x == x && p.y == y;
}

```

这段程序只有当对象具有相同的实现类时，才会产生相等的效果。否则会出现无法接受的结果，例如，让它的构造函数跟踪创建了多少实例：

```java
public class CounterPoint extends Point {
    private static final AtomicInteger counter = new AtomicInteger();

    public CounterPoint(int x, int y) {
        super(x, y);
        counter.incrementAndGet();
    }

    public static int numberCreated() {
        return counter.get();
    }
}

```

```java
public class CounterPointTest {
    // Initialize unitCircle to contain all Points on the unit circle  (Page 43)
    private static final Set<Point> unitCircle = Set.of(
            new Point( 1,  0), new Point( 0,  1),
            new Point(-1,  0), new Point( 0, -1));

    public static boolean onUnitCircle(Point p) {
        return unitCircle.contains(p);
    }

    public static void main(String[] args) {
        Point p1 = new Point(1,  0);
        Point p2 = new CounterPoint(1,  0);

        // Prints true
        System.out.println(onUnitCircle(p1));

        // Should print true, but doesn't if Point uses getClass-based equals
        System.out.println(onUnitCircle(p2));
    }
}
```

> true  false

`Point` 类使用基于 `getclass `的` equals `方法，那么不管`CounterPoint `实例的 `x `和` y` 坐标如何，`onUnitCircle `方法都会返回 `false`。

虽然没有令人满意的方法来继承一个可实例化的类并添加一个值组件，但是有一个很好的解决方案：遵循“**复合优先于继承**”。给 `ColorPoint` 一个私有的 Point 字段和一个 public 视图方法，而不是让 `ColorPoint `继承 Point，该方法返回与这个颜色点相同位置的点：

```java
// Adds a value component without violating the equals contract
public class ColorPoint {
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    /**
    * Returns the point-view of this color point.
    */
    public Point asPoint() {
        return point;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;

        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }
    ... // Remainder omitted
}

```

> 注意，你可以向抽象类的子类添加一个值组件并且不违反 equals 约定。这对于遵循 “**23条：层次优先于标签类**”中的建议而得到的类层次结构来说，这一点很重要。例如，可以有一个没有值组件的抽象类Shape、Circle子类添加了一个radius域和Rectangle子类添加了length和width域。只要不可能直接创建超类实例，前面显示的那种问题就不会发生。

- 一致性：对于任何非空的引用值 x 和 y, `x.equals(y)` 的多次调用必须一致地返回 true 或一致地返回 false，前提是不修改 equals 中使用的信息。

随着时间的推移，可能对象中的字段域的值发生变化，导致结果的变化，由此得出的结论是：**不要使equals方法依赖于不可靠的资源**

- 非空性（Non-nullity）：所有对象都不等于 null。

许多类都有相等的方法，通过显式的 null 测试来防止它：

```java
@Override
public boolean equals(Object o) {
    if (o == null)
        return false;
    ...
}

```

有的时候`equals` 方法必须首先将其参数转换为适当的类型，以便能够调用其访问器或访问其字段。在执行转换之前，方法必须使用 `instanceof `运算符来检查其参数的类型是否正确：

```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof MyType))
        return false;
    MyType mt = (MyType) o;
    ...
}

```

### 总结

综上所述，这里有一个高质量构建 equals 方法的秘诀：

- **使用 == 运算符检查参数是否是对该对象的引用。** 如果是，返回 true。这只是一种性能优化，但如果比较的代价可能很高，那么这种优化是值得的。

- **使用` instanceof `运算符检查参数是否具有正确的类型。**
- **将参数转换为正确的类型。**
- **对于类中的每个「重要」字段，检查参数的字段是否与该对象的相应字段匹配。**

> 对于类型不是 float 或 double 的基本类型字段，使用 == 运算符进行比较；
>
> 对于对象引用字段，递归调用 equals 方法；
>
> 对于 float 字段，使用 `static Float.compare(float,float)` 方法；
>
> 对于` double` 字段，使用 `Double.compare(double, double)`。`float` 和 `double` 字段的特殊处理是由于 `Float.NaN`、`-0.0f `和类似的双重值的存在而必须的；虽然你可以将 `float` 和 `double` 字段与静态方法 `Float.equals` 和 `Double.equals `进行比较，这将需要在每个比较上进行自动装箱，这将有较差的性能。
>
> 对于数组字段，将这些指导原则应用于每个元素。如果数组字段中的每个元素都很重要，那么使用 `Arrays.equals` 方法之一。

```java
package effectivejava.chapter3.item10.test;

import effectivejava.chapter3.item10.Point;

import java.util.Arrays;
import java.util.Objects;

/**
 * @Author zjm
 * @Description:
 * @Date: Created in 14:05 2021/7/28
 * @Modified By:
 */
public class Test {

    private String a;

    private float b;

    private long c;

    private double d;

    private Point point;//引用类型

    private String[] strings;

    private Point[] points;

    private int[] ints;

    private double[] doubles;

    private float[] floats;

    /*@Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Test test = (Test) o;
        return Float.compare(test.b, b) == 0 && c == test.c && Double.compare(test.d, d) == 0
                && a.equals(test.a) && point.equals(test.point) && Arrays.equals(strings, test.strings)
                && Arrays.equals(points, test.points) && Arrays.equals(ints, test.ints)
                && Arrays.equals(doubles, test.doubles) && Arrays.equals(floats, test.floats);
    }

    @Override
    public int hashCode() {
        int result = Objects.hash(a, b, c, d, point);
        result = 31 * result + Arrays.hashCode(strings);
        result = 31 * result + Arrays.hashCode(points);
        result = 31 * result + Arrays.hashCode(ints);
        result = 31 * result + Arrays.hashCode(doubles);
        result = 31 * result + Arrays.hashCode(floats);
        return result;
    }*/

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Test test = (Test) o;
        return Float.compare(test.b, b) == 0 && c == test.c && Double.compare(test.d, d) == 0
//一些对象引用字段可能合法地包含 null。为了避免可能出现 NullPointerException，请使用静态方法Objects.equals(Object, Object) 检查这些字段是否相等。
                && Objects.equals(a, test.a) && Objects.equals(point, test.point)
                && Arrays.equals(strings, test.strings) && Arrays.equals(points, test.points)
                && Arrays.equals(ints, test.ints) && Arrays.equals(doubles, test.doubles) && Arrays.equals(floats, test.floats);
    }

    @Override
    public int hashCode() {
        int result = Objects.hash(a, b, c, d, point);
        result = 31 * result + Arrays.hashCode(strings);
        result = 31 * result + Arrays.hashCode(points);
        result = 31 * result + Arrays.hashCode(ints);
        result = 31 * result + Arrays.hashCode(doubles);
        result = 31 * result + Arrays.hashCode(floats);
        return result;
    }
}

```

- equals 方法的性能可能会受到字段比较顺序的影响。为了获得最佳性能，你应该首先比较那些更可能不同、比较成本更低的字段，或者理想情况下两者都比较。

### 注意事项

- **当你覆盖 equals 时，也覆盖 hashCode。**
- **不要自作聪明。** 如果你只是为了判断相等性而测试字段，那么遵循 equals 约定并不困难。如果你在寻求对等方面过于激进，很容易陷入麻烦。一般来说，考虑到任何形式的混叠都不是一个好主意
- **不要用另一种类型替换 equals 参数声明中的对象。**

```java
// Broken - parameter type must be Object!
public boolean equals(MyClass o) {
    ...
}
```

这里的问题是，这个方法没有覆盖其参数类型为 `Object` 的 `Object.equals`，而是重载了它。即使是普通的方法，提供这样一个「强类型的」equals 方法是不可接受的，因为它会导致子类中的重写注释产生误报并提供错误的安全性。

一致使用 Override 注释将防止你犯此错误（Item-40）。这个 equals 方法不会编译，错误消息会告诉你什么是错误的：

```java
// Still broken, but won’t compile
@Override
public boolean equals(MyClass o) {
    ...
}
```

